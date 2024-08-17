---
title: 如何实现 UITabbarController 的 State Preservation?
categories:
  - iOS
date: 2016-01-19 21:58:07
tags:
  - UITabbarController
---

> [原文链接](http://blog.samwei12.cn/2016/01/19/Objective-C/UITabbarController-State-Preservation/)

最近在看**ios programming - the big nerd ranch guide** 这本书,其中第24章介绍了如何使用系统接口来实现 State Restoration. 示例部分介绍的是如何针对 `UINavigationController` 来进行保存和还原状态, 然后额外的练习题部分是 `UITabbarController` 的状态保存和恢复,可是在这里却一直遇到问题， 导致程序返回时`UITabbarController`始终无法还原状态，本文记录下如何使用State Restoration和`UITabbarController`所需的额外处理。

<!--more-->

首先, 我们需要告诉系统我们想要实现状态的保存和恢复, 在 AppDelegate 中实现如下两个方法:

```objc
- (BOOL)application:(UIApplication *)application shouldSaveApplicationState:(NSCoder *)coder {
    return YES;
}

- (BOOL)application:(UIApplication *)application shouldRestoreApplicationState:(NSCoder *)coder {
    return YES;
}
```

其次, 我们需要给每个页面赋予一个独有的`restorationIdentifier`, 两个子页面还需要设置`restorationClass`, 用于还原状态时重新生成view controller. 代码如下:

```objc
HypnosisViewController *hvc = [HypnosisViewController new];
ReminderViewController *rvc = [ReminderViewController new];
UITabBarController *tabVC = [[UITabBarController alloc] init];
tabVC.viewControllers = @[hvc, rvc];

tabVC.restorationIdentifier = NSStringFromClass([tabVC class]);

...
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {
    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
    if (self) {
        self.tabBarItem.title = @"Hypno";
        self.tabBarItem.image = [UIImage imageNamed:@"Hypno"];

        self.restorationIdentifier = NSStringFromClass([self class]);
        self.restorationClass = [self class];
    }
    return self;
}
...
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {
    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
    if (self) {
        NSLog(@"%s line:%d",__FUNCTION__, __LINE__);
        self.tabBarItem.title = @"Reminder";
        self.tabBarItem.image = [UIImage imageNamed:@"Time"];

        self.restorationIdentifier = NSStringFromClass([self class]);
        self.restorationClass = [self class];
    }
    return self;
}
```

接着,我们需要在子页面中实现`UIViewControllerRestoration`协议,同时还要重写` UIViewController`自带的用于 Restoration 的两个方法, 下面以`HypnosisViewController`为例,这里在收到系统需要保存状态消息时,写入当前` textField`的值,并且在还原状态时读取,确保用户下次进入程序时仍能还原回之前的场景:

```objc
+ (UIViewController *)viewControllerWithRestorationIdentifierPath:(NSArray *)identifierComponents coder:(NSCoder *)coder {
    UIViewController *vc = [self new];
    return vc;
}

- (void)encodeRestorableStateWithCoder:(NSCoder *)coder {
    [coder encodeObject:self.textField.text forKey:@"textField"];
    [super encodeRestorableStateWithCoder:coder];
}

- (void)decodeRestorableStateWithCoder:(NSCoder *)coder {
    self.textField.text = [coder decodeObjectForKey:@"textField"];
    [super decodeRestorableStateWithCoder:coder];
}
```

最后, 我们只提供了两个子页面如何进行还原, 还需要处理下` UITabbarController`, 如果我们不提供 Controller 的`restorationClass`属性, 我们可以在`application:viewControllerWithRestorationIdentifierPath:coder:`中再次处理部分页面的恢复事件,代码如下:

```objc
UIViewController *vc = [UITabBarController new];
vc.restorationIdentifier = [identifierComponents lastObject];
//因为仅当该 Controller 为 UITabbarController 时, identifierComponents数组才会只有一个值
if (identifierComponents.count == 1) {
    self.window.rootViewController = vc;
}
return vc;
```

至此,我们理论上算是已经完成了所有的保存和恢复事件,起码书里面针对`UINavigationController`的示例代码部分仅包含这些步骤就已经实现了这个功能,那么,`UITabbarController`是不是一样呢?

## 问题

运行程序,为了检验Restoration 的效果,我们进入第二个页面,并且设置一个日期,之后进入后台或者停止调试触发保存事件,当我们再次启动程序时, 可以看到这个页面一闪而过:

![](http://7xlmda.com1.z0.glb.clouddn.com/2016-01-19_22-49-48.png)

之后就变为了这样:

![](http://7xlmda.com1.z0.glb.clouddn.com/2016-01-19_22-49-13.png)

事实证明,我们的 `UITabbarController` 并没有很好地还原为之前的状态,甚至页面变成全空的了,为什么会变成这样呢?

## 探究

首先,通过启动程序时的 loading 图可以确定,我们的程序确实已经保存下了之前退出时候的页面状态,后面的白屏证实还原状态这里出了问题,那么即使`UITabbarController`无法很好地还原,为什么再次启动后会连子页面都不见了呢?

### View Debug

这里, 我们首先想到使用 ios8以后引入的View Debugging来查看 View 层级, 如下图:

![](http://7xlmda.com1.z0.glb.clouddn.com/2016-01-20_22-26-09.png)

这里我们可以看到, View 层级中,并没有任何子页面,仅仅包含一个 UITabbar 而已,也就是说,我们的两个子页面根本没有添加到 UITabbarController 中, 这又是怎么一回事呢?

### 查阅文档

我们看下苹果官方文档对这里是如何描述的:

> In iOS 6 and later, if you assign a value to this view controller’s restorationIdentifier property, it preserves a reference to the view controller in the selected tab. At restore time, it uses the reference to select the tab with the same view controller.

看起来好像没有任何问题,只是简单地说了指定 UITabbarController 的`restorationIdentifier`后,它会记录下当前选中的子页面,然后在下次启动的时候恢复过来. 接着看下去,[App Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforImplementingYourApp/StrategiesforImplementingYourApp.html)是这么说的:

> Although UIKit helps restore the individual view controllers, it does not automatically restore the relationships between those view controllers. Instead, each view controller is responsible for encoding enough state information to return itself to its previous state. For example, a navigation controller encodes information about the order of the view controllers on its navigation stack. It then uses this information later to return those view controllers to their previous positions on the stack. Other view controllers that have embedded child view controllers are similarly responsible for encoding any information they need to restore their children later.
> Note: Not all view controllers need to encode their child view controllers. For example, tab bar controllers do not encode information about their child view controllers. Instead, it is assumed that your app follows the usual pattern of creating the appropriate child view controllers prior to creating the tab bar controller itself.

这两段话大致是说, UIKit 虽然帮我们处理了恢复单独页面的事情,但是并不会帮我们恢复页面之间的逻辑关系, 那些包含多个子页面的 Controller 需要自己记录子页面间的逻辑关系并且在恢复时处理它们. 这里还专门说了 UINavigationController 会记录下页面的层级关系并且在再次创建时恢复它,而第二段话说 UITabbarController 却不会记录子页面的顺序, 需要我们自己进行处理. 同时,我们可以在恢复页面时候自行变更 UITabbarController 子页面的顺序.

## 解决问题

有了以上知识点,我们就知道了如何还原一个 UITabbarController 页面了,除了赋值`restorationIdentifier`以外,我们还需要在还原时候自行添加子页面.

AppDelegate.m

```objc
- (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
    self.window.backgroundColor = [UIColor whiteColor];

// 1.
    HypnosisViewController *hvc = [HypnosisViewController new];
    ReminderViewController *rvc = [ReminderViewController new];
    UITabBarController *tabVC = [[UITabBarController alloc] init];
    tabVC.viewControllers = @[hvc, rvc];

    tabVC.restorationIdentifier = NSStringFromClass([tabVC class]);

    self.window.rootViewController = tabVC;

    return YES;
}
```
这样,每次程序启动时,即创建一个新的 UITabbarController, 同时赋予它两个子页面,


新增了两处代码,用于指定新建的 UITabbarController 的子页面, 之后对两个子页面的`+ (UIViewController *)viewControllerWithRestorationIdentifierPath:(NSArray *)identifierComponents coder:(NSCoder *)coder`方法改动如下:

HypnosisViewController.m

```objc
+ (UIViewController *)viewControllerWithRestorationIdentifierPath:(NSArray *)identifierComponents coder:(NSCoder *)coder
{
    // First object in the array stores the root view controller, which is the tabBarController
    UITabBarController *rootViewController = (UITabBarController *)[UIApplication sharedApplication].delegate.window.rootViewController;
    
    return rootViewController.viewControllers[0];
}
```

ReminderViewController.m

```objc
+ (UIViewController *)viewControllerWithRestorationIdentifierPath:(NSArray *)identifierComponents coder:(NSCoder *)coder
{
    // First object in the array stores the root view controller, which is the tabBarController
    UITabBarController *rootViewController = (UITabBarController *)[UIApplication sharedApplication].delegate.window.rootViewController;
    
    return rootViewController.viewControllers[1];
}
```
用于告诉系统,这两个子页面不需要再次创建,直接返回当前 UITabbarController 子页面即可.
再次运行程序,发现已经能够正常返回到第二个页面了,大功告成!

## 扩展

之前文档中有写到, UITabbarController 仅保存了当前选中的页面的指针,而没有保存页面的顺序, 那么,也就是说,我们可以在还原时候重新自定义tab 的顺序, 对代码进行以下改动:

1. AppDelegate.md

```objc
- (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
...
    tabVC.viewControllers = @[rvc, hvc];
...
    return YES;
}
```

2. HypnosisViewController.m

```objc
+ (UIViewController *)viewControllerWithRestorationIdentifierPath:(NSArray *)identifierComponents coder:(NSCoder *)coder {
...
    return rootViewController.viewControllers[1];
}
```

3. ReminderViewController.m

```objc
+ (UIViewController *)viewControllerWithRestorationIdentifierPath:(NSArray *)identifierComponents coder:(NSCoder *)coder {
...
    return rootViewController.viewControllers[0];
}
```

再次运行, 得到:

![](http://7xlmda.com1.z0.glb.clouddn.com/2016-01-20_23-42-29.png)

我们成功的在还原页面状态时候变更了页面顺序,而且UITabbarController 仍然选中为之前保存的状态, 大功告成!

## 参考文档

* [UITabBarController Class Reference
](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITabBarController_Class/)
* [Help needed for silver challenge](http://forums.bignerdranch.com/viewtopic.php?f=505&t=8402)
* [App Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/StrategiesforImplementingYourApp/StrategiesforImplementingYourApp.html)
* [http://aplus.rs/2013/state-restoration-in-ios-6-without-storyboards/](http://aplus.rs/2013/state-restoration-in-ios-6-without-storyboards/)


