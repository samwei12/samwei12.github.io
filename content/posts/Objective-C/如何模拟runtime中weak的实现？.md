---
title: 如何实现ARC中weak功能？
date: 2016-03-09 11:27:09
categories: 
  - iOS
tags: 
  - Runtime 
  - Objective-C 
  - ARC
---

> [原文链接](http://blog.samwei12.cn/2016/03/09/Objective-C/%E5%A6%82%E4%BD%95%E6%A8%A1%E6%8B%9Fruntime%E4%B8%ADweak%E7%9A%84%E5%AE%9E%E7%8E%B0%EF%BC%9F/)

我们都知道ARC中`weak`与`assign`或者说`unsafe_unretained`最大的不同就是设置`weak`属性后，系统会在对象被释放后自动将指向对象的指针置为`nil`，而`assign`则会产生一个悬空指针，那么系统是如何实现这一机制呢？我们能否自己模拟系统对`weak`的实现呢？

<!--more-->

通过查看runtime源码中`objc-accessors.h`和`objc-weak.h`部分，我们大概可以了解系统针对`weak`的实现方式与`strong`或者`copy`的实现方式是不同的。
对于注册为`weak`的对象，系统会以`weak`指向的对象内存地址作为key，将之放入到一个hash表之中，当此对象的引用计数为0时会调用dealloc，之后遍历hash表中此key所对应的对象，将之置为nil，关于这部分详细内容可以自行查看源代码，在此不进行扩展。我们这里重点介绍下如何自行模拟系统这一机制。

## 分析

首先我们回想一下`weak`机制的几个步骤，其中真正与`assign`不相同的部分在于当对象调用`dealloc`之后对对象指针置空操作。那么我们很容易联想到Objective-C中关联对象的使用，关联对象也是在对象调用`dealloc`之后被移除，这样我们是否可以通过对对象添加一个关联对象来模拟`weak`的实现呢？

## 探究

下面我们通过代码来说明下如何自行实现一个类似`weak`机制的对象管理机制，本文中所用到的代码存放在[https://github.com/samwei12/WeakDemo](https://github.com/samwei12/WeakDemo) ：

### `unsafe_unretained`和`weak`具体使用中的差异

1. 首先，创建一个`ClassA`类，
2. 创建一个`ClassB`类，并在里面添加一个测试方法`print`，用于等下向`ClassB`实例对象发送消息，确认对象是否仍在使用

    ```objc
    - (void)print {
        NSLog(@"Object is %@", self);
    }
    ```

3. 给`ClassA`添加一个`ClassB`对象属性，并设置为`unsafe_unretained`

    ```objc
    @property (nonatomic, unsafe_unretained) ClassB *objectB;
    ```

4. 在main函数中添加如下代码：

    ```objc
    @autoreleasepool {
        ClassA *instanceA = [ClassA new];

        ClassB *instanceB = [ClassB new];
        instanceA.objectB = instanceB;

        [instanceA.objectB print];
            // release instanceB
        instanceB = nil;


        [instanceA.objectB print];
    }

    // prevent process be killed immediately
    sleep(3);
    return 0;
    ```

5. 运行一下代码，不出意外的话，程序会挂掉，因为`instanceA.objectB`所指向的内存已经在`instanceB = nil;`时候被释放掉，`instanceA.objectB`仅指向一个悬空指针：

    ```objc
    2016-03-09 15:25:10.427 WeakDemo[98402:1613532] Object is 0x7fa080d0f100
    2016-03-09 15:25:10.432 WeakDemo[98402:1613532] Object is 0x7fa080d0f100

    Process finished with exit code 139
    ```
    当然，也有可能不会挂，这取决于执行`print`函数时，`instanceB`所在的内存是否完全释放掉。

6. 如果将`ClassA`中的属性改为`@property (nonatomic, weak) ClassB *objectB;`则不会出现crash，这就是`weak`的作用了，`objectB`对象如果被释放掉，则该指针变为`nil`，而向`nil`发送消息是不会出现问题的。


### 如何给`unsafe_unretained`添加`weak`功能

下面我们就使用`unsafe_unretained`来一步一步模拟`weak`修饰符：

1. 创建一个`DeallocBlock`类，代码如下：

    ```objc
    typedef void (^voidBlock)();
    @interface DeallocBlock : NSObject
    
    @property(nonatomic, copy) voidBlock block;
    
    - (instancetype)initWithBlock:(voidBlock)block1;
    
    @end
    
    @implementation DeallocBlock
    - (instancetype)initWithBlock:(voidBlock)block1 {
        self = [super init];
        if (self) {
            _block = [block1 copy];
        }
        return self;
    }
    
    
    - (void)dealloc {
        if (_block) {
            NSLog(@"DeallocBlock dealloc!");
            _block();
        }
    }
    @end
    ```
    覆盖它的`dealloc`函数，这样就实现了该对象被释放时执行外部传入的`block`功能。
2. 添加一个`NSObject`的category，如下：

    ```objc
    #import <Foundation/Foundation.h>
    #import "DeallocBlock.h"
    #import <objc/runtime.h>
    @interface NSObject (deallocBlock)
    - (void)runBlockOnDealloc:(voidBlock)block;
    @end
    
    @implementation NSObject (deallocBlock)
    - (void)runBlockOnDealloc:(voidBlock)block {
        if (block) {
            DeallocBlock *deallocBlock = [[DeallocBlock alloc] initWithBlock:block];
            objc_setAssociatedObject(self, _cmd, deallocBlock, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
        }
    }
    @end
    ```
    这里主要实现的功能是，给所有的`NSObject`对象添加一个传入`block`的方法，这个方法的作用就是在其中根据传入的`block`生成一个`DeallocBlock`实例，并使用关联对象将这个实例关联到自身。之后当这个对象被释放时候，会自动移除所有的关联对象，也就会触发`DeallocBlock`实例的`dealloc`方法，执行我们传入的`block`。

3. 接下来，我们重载`ClassA`的`setObjectB`方法：

    ```objc
    - (void)setObjectB:(ClassB *)objectB {
        _objectB = objectB;
        // 仅对objectB != nil case做处理
        if (_objectB) {
            [_objectB runBlockOnDealloc:^{
                NSLog(@"_objectB dealloc");
                _objectB = nil;
            }];
        }
    }
    ```
    这段代码主要作用就是给`_objectB`添加一个`deallocBlock`,这样，如果`_objectB`所指向的内存已经被释放掉，则会调用我们传递进去的`block`来将此悬空指针置为空

4. 再次运行程序：

    ```objc
    2016-03-09 15:53:36.881 WeakDemo[99270:1639867] Object is 0x7fad79c0f310
    2016-03-09 15:53:36.882 WeakDemo[99270:1639867] DeallocBlock dealloc!
    2016-03-09 15:53:36.882 WeakDemo[99270:1639867] _objectB dealloc
    
    Process finished with exit code 0
    ```
    由此可知，在`main`函数中执行`instanceB = nil;`之后，开始执行`deallocBlock`，之后由于`instanceA.objectB`已经为`nil`，`print`方法不执行操作。

## 总结

至此，我们已经实现了自己模拟系统`weak`机制的运行，给一个本会产生悬空指针的`unsafe_unretained`修饰符加上了`weak`的功能。大家可以自行尝试[https://github.com/samwei12/WeakDemo](https://github.com/samwei12/WeakDemo),如果有任何问题，欢迎指正。

## 参考文档

* [https://www.opensource.apple.com/source/objc4/](https://www.opensource.apple.com/source/objc4/)
* [Fun With the Objective-C Runtime: Run Code at Deallocation of Any Object](http://blog.slaunchaman.com/2011/04/11/fun-with-the-objective-c-runtime-run-code-at-deallocation-of-any-object/)


