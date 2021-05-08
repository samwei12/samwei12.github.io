---
title: AVCaptureSession部分用法
tags:
  - AVCaptureSession
categories:
  - iOS
date: 2015-09-21 13:58:46
---

> [原文链接](http://blog.samwei12.cn/2015/09/21/Objective-C/AVCaptureSession%E9%83%A8%E5%88%86%E7%94%A8%E6%B3%95/)

## AVCaptureSession阻塞主线程问题

前阵子程序中出现了一个奇怪的 bug，在 iOS 系统上，页面弹出的时候会卡很久，相机始终黑屏，大概6-7秒钟，跟踪具体每个步骤花费时间的时候发现在`viewWillDisappear:`中开销最大，这其中只调用了一个相机关闭的代码：

```objc
if ([[self.avCameraManager session] isRunning]) {
			[[self.avCameraManager session] stopRunning];
}
```

仔细看了文档之后，发现问题出在` stopRunning`这里，
<!--more-->
苹果文档描述如下：

> Clients invoke -stopRunning to stop the flow of data from inputs to outputs connected to 
    the AVCaptureSession instance.  This call blocks until the session object has completely
    stopped.

重点是这个函数在 session 完全停止下来之前会始终阻塞线程， 同样的，在` startRunning`中：

> Clients invoke -startRunning to start the flow of data from inputs to outputs connected to 
    the AVCaptureSession instance.  This call blocks until the session object has completely
    started up or failed.  A failure to start running is reported through the AVCaptureSessionRuntimeErrorNotification
    mechanism.

因此这里必须放在后台线程中处理，否则，就会导致界面不响应，iOS8之后应该在这里做了优化，即使放在主线程做也没有很卡顿的现象，但 iOS7中，尤其是测试设备为4s，界面卡死问题很严重。

开启和关闭相机部分代码改为：

```objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
		DDLogDebug(@"Function: %s,line : %d 开启相机",__FUNCTION__,__LINE__);
		[[self.avCameraManager session] startRunning];
	});
	
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void) {
		DDLogDebug(@"Function: %s,line : %d 关闭相机",__FUNCTION__,__LINE__);
		if ([[self.avCameraManager session] isRunning]) {
			[[self.avCameraManager session] stopRunning];
		}
	});
```

这里除了使用系统提供的队列以外还可以自己创建 FIFO 类型后台线程进行管理，包括切换前后摄像头、改变闪光灯模式、切换拍照和录像等，都可以放入子线程操作。

## 相机前后台切换问题

另一个问题与前后台切花相关，项目中在程序进入后台时有这么一段代码：

```objc
[self performSelector:@selector(startRunningSession) withObject:nil afterDelay:0.5];
```

当时我很奇怪，这里为什么要加延时呢？进入到前台时不是应该直接开启相机吗？于是我直接把这里改成了：

```objc
[self startRunningSession];
```

没过多久，问题出现了，多次切换前后台之后发现，相机始终黑屏，这时，进入后台再回来，问题解决了，仔细加了 log 之后发现，出现黑屏的时候，确实也正确调用了启动 session，但是并没有收到系统相机启动成功的消息，反而收到了两次相机关闭的通知。

那么之前代码中加了0.5秒延时的原因也就清楚了，是为了等待系统关闭相机之后，再调用开启相机，以免相机启动失败，但这种方式真的合理吗？其实并没有解决本质问题，如果0.5秒钟的时间并没有完全关闭系统相机呢？这里仍然会出现黑屏问题。

于是继续查看系统文档，发现系统的 sampleCode AVCam-iOS中并没有专门用于前后台处理的逻辑，原来这里并不需要我们自己手动管理相机，系统会自动根据程序状态来判断是否需要关闭或者开启相机，如果按 home 进入`UIApplicationStateBackground` 状态，那么系统自动关闭相机并在进入`UIApplicationStateActive`状态时开启相机；如果是进入`UIApplicationStateInactive`状态，例如，双击 home 调出任务管理器，这时，相机并没有被关闭，仍然能够看到 preview 画面。

我们在程序切换前后台时，仅需要捕获相继开启或者关闭的通知来刷新界面即可，否则可能会由于快速开启切换前后台导致系统相机执行命令错乱，无法正确启动相机。


## 总结

* AVCaptureSession 中绝大部分操作需要在后台线程完成，最好使用一个 FIFO 的队列来进行操作
* 前后台切换时，无需手动管理 CaptureSession
