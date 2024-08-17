---
title: 如何取消 UIView 动画？
tags:
  - Animation
  - UIView
categories:
  - iOS
date: 2015-09-09 14:28:02
---

> [原文链接](http://blog.samwei12.cn/2015/09/09/Objective-C/%E5%A6%82%E4%BD%95%E5%8F%96%E6%B6%88UIView%E5%8A%A8%E7%94%BB/)

最近项目中有一个需求是需要手动点击相机对焦，这里由于相机对焦部分需要一个类似于系统对焦框一样的缩放动画，同时动画时长为0.3秒，因此这里就有一个很普遍的需求，如果用户在0.3秒内继续点击对焦会怎么样？

动画部分代码很简单，如下：

```objc
self.transform = CGAffineTransformMakeScale(2.0f, 2.0f);
    [UIView animateWithDuration:0.3f delay:0.0f options:UIViewAnimationOptionCurveLinear
                     animations:^{
                         self.transform = CGAffineTransformIdentity;
                     }
                     completion:^(BOOL finished){
						 if (finished) {
							 [self hideFocus];
						 }
                     }];
```

对焦框在0.3秒内进行两倍缩小到正常尺寸的一个动画，之后隐藏。

<!--more-->

## 旧代码分析

以前的代码在这部分的处理大致是这样的，

```objc
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
    [NSObject cancelPreviousPerformRequestsWithTarget:self selector:@selector(touchFocus:) object:_touches];
    
    UITouch *touch = [_touches anyObject];
    CGPoint touchPoint = [touch locationInView:self.view];


    [self performSelector:@selector(touchFocus:) withObject: touches afterDelay:0.3f];
    
- (void)touchFocus:(NSSet *)touches {
    [NSObject cancelPreviousPerformRequestsWithTarget:self selector:@selector(touchFocus:) object:touches];
    
    // 以下为具体对焦框显示部分
    ...
}
```

可以看出，代码逻辑大致为，截获屏幕点击事件，之后加入一个0.3秒的延时方法，如果用户0.3秒内再次点击对焦，则取消上一次的对焦事件；否则，执行对焦框动画。

这个逻辑看似解决了用户快速点击对焦的问题，实际上如果用户真正快速点击屏幕时，会出现一个很奇怪的现象，就是对焦框始终存在，并且不会变换位置，等到停止点击屏幕后，对焦框才会响应最新的触控位置进行动画，另一个问题就是对焦始终存在迟滞感，因为点击屏幕后需要等待0.3秒才能看到动画。

## 解决方案

那么有什么更好的办法呢？系统难道没有提供终止动画的接口嘛？当然不是，其实只需要在用户再次点击屏幕时候调用`[_focusView.layer removeAllAnimations];`即可，其中具体的 view 为正在做动画的 view。

这样做还有另外一个好处，再也不需要烦人的滞后感了，屏幕对焦始终跟手，同时直接终止动画，里面的 completion 都会照常调用，不存在副作用，很好地解决了这个问题。

## 新引入问题

最近遇到了一个新引入的问题，发现对焦框无法隐藏，经过调查，问题出在这里：

```objc
if (finished) {
    [self hideFocus];
}
```

如果是直接使用` removeAllAnimations`接口停止动画的话，此处的` finished`参数有可能返回 NO，导致这段代码产生问题，所以这里如果中止动画，则需要重新进行判定。

## 总结

* 学会了 `removeAllAnimations`接口用法

尽管这个是很基础的系统 API 调用，但是我之前一直都不知道，在此记录下来，希望能给予其他人些许帮助。

