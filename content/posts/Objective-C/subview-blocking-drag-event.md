---
title: NSView subview blocking drag/drop event
date: 2016-01-26 13:41:57
categories:
- Mac
tags:
- Cocoa
- NSView
---

> [原文链接](http://blog.samwei12.cn/2016/01/26/Objective-C/subview-blocking-drag-event/)

近期在Mac项目中有一个处理鼠标拖拽事件的需求, 大致处理流程是这样的:

1. 从 NSView 继承得到一个子类
2. 覆盖处理拖拽事件相关方法
3. 注册拖拽事件

<!--more-->

开始的时候一切都很正常,直到某次发现拖拽到屏幕边缘响应较为灵敏,而**拖拽到屏幕中间则不响应事件**,APP页面大致是这样的:

![](http://7xlmda.com1.z0.glb.clouddn.com/2016-01-28_19-27-12.png)

## 问题分析

开始我一直以为是系统事件不太灵光, 后来发现只有在拖动到屏幕中间时候出现这个问题, 这个现象十分奇怪, 后来联想到页面中间有一张图片，猜测是否是因为图片截获了拖拽事件。

那么如何验证呢？其实也很简单，在自定义的NSView 中重载`draggingEnded：`方法，同时添加 log，发现鼠标拖拽到页面中间时， 该方法就不再被调用。因此可以得出结论，在自定义的 `NSView` 中添加的`NSImageView`作为 subview 会阻止拖拽事件的传递，原因是` NSImageView`本身可以处理拖拽事件，于是截获了本该自定义`NSView`处理的消息。

## 解决方案

在网上搜索了一下这个问题，发现已经有人给出了解决方案，具体链接：[http://stackoverflow.com/questions/5892464/cocoa-nsview-subview-blocking-drag-drop](http://stackoverflow.com/questions/5892464/cocoa-nsview-subview-blocking-drag-drop)

提供的解决方案有以下几种：

1. 不使用 `NSImageView`，而是采用自定义` NSView`的方式，然后重写`drawRect：`方法进行图片渲染。这个办法十分笨拙，并且不太方便。
2. 调用`unregisterDraggedTypes `方法，这样` NSImageView`就不会监听拖拽事件了，完美解决。



## 参考文档

* [http://stackoverflow.com/questions/5892464/cocoa-nsview-subview-blocking-drag-drop](http://stackoverflow.com/questions/5892464/cocoa-nsview-subview-blocking-drag-drop)
* [http://stackoverflow.com/questions/4782636/nsview-subviews-interrupting-drag-operation](http://stackoverflow.com/questions/4782636/nsview-subviews-interrupting-drag-operation)


