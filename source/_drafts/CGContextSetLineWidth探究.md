title: CGContextSetLineWidth探究
date: 2015-09-09 16:31:15
tags:
- CoreGraphics
categories:
- iOS
---

前阵子遇到了一个`CGContextSetLineWidth `绘制问题，使用该函数绘制了一个矩形边框，结果测试发现，在6plus 上和在其他设备上表现不一致，像素差的较多，于是网上查了下，这里大部分是说`CGContextSetLineWidth `函数接收的参数是像素点，也就是 pt，同时 iOS 上允许绘制的最小像素为2px，想要更小的话则需要设置抗锯齿参数，那么究竟是不是这样呢？
> 本文所采用的测量工具为 Mac 下的 Xscope

<!--more-->

## CGContextSetLineWidth参数单位？

针对这个函数传入的宽度单位，苹果官方文档是这么描述的
> A value, in user space units, that is greater than zero. This value does not affect the line width values in the current graphics state.

这里的`user space units`,应该就是通常所说的像素点 pt 了，同时网上查到的很多帖子也是这么说的，那么实际上是不是这样呢？

测试代码如下 ：

```objc
- (void)drawRect:(CGRect)rect {
    // Drawing code
	CGContextRef currentContext = UIGraphicsGetCurrentContext();
	CGContextSetStrokeColorWithColor(currentContext, [UIColor blueColor].CGColor);
	CGContextStrokeRectWithWidth(currentContext, rect, 2);
//	CGContextSetShouldAntialias(currentContext, NO);
	CGContextStrokePath(currentContext);
}

// VC 中使用

self.testView = [CGContextTestView new];
self.testView.frame = CGRectMake(100, 100, 100, 100);
self.testView.backgroundColor = [UIColor whiteColor];
[self.view addSubview:self.testView];
```

即，绘制一个`100pt*100pt`的矩形，边框为`2pt*2pt`，这段代码在双倍分辨率下，预期效果为一个边框`4px`，宽高为`200px*200px`的矩形，三倍分辨率，也就是6plus 上则为一个边框`6px`，宽高`300px*300px`的矩形



现在，我们改变代码，`CGContextSetLineWidth `转为传入整数参数，即：

```objc
- (void)drawRect:(CGRect)rect {
    // Drawing code
	CGContextRef currentContext = UIGraphicsGetCurrentContext();
	CGContextSetStrokeColorWithColor(currentContext, [UIColor blueColor].CGColor);
	CGContextStrokeRectWithWidth(currentContext, rect, 2);//原先是0.8
	CGContextStrokePath(currentContext);
}
```


# CGContextSetLineWidth 绘制探究

## CGContextSetLineWidth 传入非整数参数

首先我们测验下， iOS 的像素取舍机制，即非整数像素时如何进行绘制，代码如下：


这里我们绘制了一个`100pt *100 pt`的矩形，同时边框设置为`0.8pt`，在正常双倍分辨率屏幕下，预期效果为矩形 `200px * 200px`，边框宽度为`1px`或者`2px`，
在4s 设备上，结果如下：

图1-1 双倍分辨率下，宽度设置为0.8pt 效果
![](http://7xlmda.com1.z0.glb.clouddn.com/CGContext_001.png)

那么这里可以推测，iOS并没有采取四舍五入的方式来绘制像素，而是直接抹去尾数，将本该是`1.6px`的边框绘制为了`1px`，也并不是网上看到的几篇文章中提到的 iOS 最小绘制`2px`，需要设置抗锯齿才能得到`1px`。

同样的代码在6p上结果如下：

图1-2 三倍分辨率下，宽度设置为`0.8pt`效果图

![](http://7xlmda.com1.z0.glb.clouddn.com/CGContext_002.png)

得到的是一个`296px*296px`的矩形，也证实了上面的推想，CGContext 的像素绘制是向下取整的，因为预期结果为`0.8pt * 3 = 2.4px`的边框，结果得到的是`2px`的边框。

> 结论：CGContextSetLineWidth 中，非整数像素的绘制采用的是向下取整的方式。



继续在双倍分辨率和三倍分辨率下分别测试，

# 参考链接
* [iphone CGContextSetLineWidth 画线的问题](http://blog.csdn.net/jxncwzb/article/details/6267154)
* [IOS CGContextSetLineWidth无法设置1像素线宽？](http://my.oschina.net/lych0317/blog/126215)


