---
layout:     post
title:      iOS CALayer介绍
subtitle:   介绍有关CALayer相关的基础知识及相关子类的使用
date:       2018-11-19-iOS 
author:     LOLITA0164
header-img: img/post-bg-layer.jpg
catalog: true
tags:
    - iOS
    - CoreAnimation
---

## 声明

该篇文章的内容参考自 [iOS核心动画高级技巧](https://zsisme.gitbooks.io/ios-/content/index.html) 一文，非常感谢其作者和中文版的作者，让我能够相对系统的学习 CoreAnimation 的知识，我受益匪浅，再次感谢。

如果有兴趣的小伙伴可以访问其网站，详细的，完整的学习 CoreAnimation。

## Core Animation 介绍

Core Animation ，核心动画，似乎第一次看到这个名字的人都会认为这是一个和动画相关的库，但是实际上，做动画只是 Core Animation 特性中的一种。由于做动画是依附于某种介质（Layer图层）的，因此可以猜想得到，该类中包含了Layer相关的东西。

Core Animation是一个复合引擎，它的职责就是尽可能快地组合屏幕上不同的可是内容，这个内容是被分解成独立的图层，存储在一个叫做*图层树*的体系中，这个体系形成了**UIKit**以及在iOS应用程序中你所看到的一切基础。

## 图层

那么图层是什么呢？它和我们 iOS 日常开发使用到的 UIView 有什么区别呢？

#### 视图（UIView）和图层（CALayer）

- UIView

视图就是在屏幕上显示的一块矩形块（如图片、文字等），它能够响应用户触摸的手势等操作，也可以支持基于 Core Graphics 绘图，可以做仿射变换（transform，平移、旋转等）。在视图的层级关系中可以相互嵌套，一个视图可以管理它的子视图。

- CALayer

CALayer 概念上和 UIView 类似，同样是一些被层级关系树管理的矩形块，同样也可以包含图片、文字等内容，可用来做动画和变换，可以管理子图层等功能，但是和 UIView 最大的不同是 CALayer 不会去处理用户的交互事件。

- UIView vs CALayer

每个 UIView 都拥有一个 CALayer 实例的图层，而 UIView 的职责就是创建并管理这个图层。实际上这个依附在视图上的图层才是真正用来在屏幕上显示和做动画的，UIView 仅仅是对 CALayer 的一个封装，提供了响应用户手势触摸等相关的功能，以及 Core Animation 底层方法的高级接口（UIView的动画块）。

#### 使用图层的必要性

在大部分的情况下，我们可通过苹果封装的 UIView 提供的高级 API 就可以完成动画等功能，但是高度封装伴随着不够灵活的缺陷，但我们想要做一些更底层的变化时，UIView 又缺乏相应的接口功能，这时候就需要我们接触到 CALayer 图层了，例如设置阴影、圆角、边框等、做3D变换、非矩形范围、不规则遮罩等等。

#### 简单使用图层

每当你创建一个视图时，系统都默认为你关联了一个图层，可以称其为关联图层，为了演示图层的使用，我们不对关联层操作，而是新创建一个图层。

我们可以直接创建 CALayer 对象，将其添加到视图关联的图层上。

示例：我们创建一个蓝色背景图层，将添加到视图的关联图层上。

```
// 创建子图层
CALayer* subLayer = [CALayer new];
// 布局
subLayer.frame = CGRectMake(0, 0, 150, 150);
subLayer.position = self.view.center;
// 设置颜色
subLayer.backgroundColor = UIColor.blueColor.CGColor;
// 添加到视图关联图层上
[self.view.layer addSublayer:subLayer];
```

![使用图层](https://ws4.sinaimg.cn/large/006tNbRwgy1fxd6gpbeh0j30s60f6glf.jpg)

## 寄宿图

在上一节中，我们演示了图层的简单实用示例：我们给该图层填充上了颜色背景，但是如果仅仅是填充颜色的话，未免有点太单调了，下面我们来给该图层填充一些其他的东西吧。

#### contents

CALayer 有一个叫 contents 的属性，在 Mac OC 中，它对 CGImage 和 NSImage 类型的值都起到作用，但是在 iOS 中，你如果将 UIImage 的值赋于它，虽然可以通过编译，但是只能得到一个空白内容。不过好在 UIImage 有一个 CGImage 属性，它返回一个 “CGImageRef” 对象，它是一个 Core Foundation 类型，你需要使用 bridge 关键字将其转换为 Cocoa 类型。就像下面：

```
layer.contents = (__bridge id)image.CGImage;
```

我们修改一个刚刚的代码

```
// 创建子图层
CALayer* subLayer = [CALayer new];
// 布局
subLayer.frame = CGRectMake(0, 0, 150, 150);
subLayer.position = self.view.center;
// 设置颜色
subLayer.backgroundColor = UIColor.groupTableViewBackgroundColor.CGColor;
// 设置 contents
subLayer.contents = (__bridge id)([UIImage imageNamed:@"snowman"].CGImage);
// 添加到视图关联图层上
[self.view.layer addSublayer:subLayer];
```

![效果图](https://ws2.sinaimg.cn/large/006tNbRwgy1fxd7goavbyj30qy0eomx3.jpg)

这样，我们通过contents为自己创建了一个具有 UIImageView 功能的图层了呢。如果该图层是某个 UIView 视图的关联层，那么是否可以认为我们自己创建了一个 UIImageView 呢？

#### contentGravity

在使用 UIImageView 中，我们经常与到图片和图片控件的大小不一致的情况，这就导致我图片看起来被压缩、拉伸的情况（就如上面情况），这时候我们通常使用 UIView 的 contentMode 属性将图片设置为比较合适的填充模式，让图片变得不是那么的“奇怪”。

```
imageView.contentMode = UIViewContentModeScaleAspectFit;
```

与之对应的，图层中有一个 contentsGravity 属性，它是一个 NSString 类型。实质上，视图的 contentMode 就是图层 contentsGravity 的一个封装。contentsGravity可选的常量值有以下一些：

- kCAGravityCenter
- kCAGravityTop
- kCAGravityBottom
- kCAGravityLeft
- kCAGravityRight
- kCAGravityTopLeft
- kCAGravityTopRight
- kCAGravityBottomLeft
- kCAGravityBottomRight
- kCAGravityResize
- kCAGravityResizeAspect
- kCAGravityResizeAspectFill

我们来试着修改一些图层的“填充模式”，在之前的代码基础上添加以下代码：

```
// 设置填充模式
subLayer.contentsGravity = kCAGravityResizeAspect;
```

![效果图](https://ws1.sinaimg.cn/large/006tNbRwgy1fxd7voe0qwj30r60ew0sn.jpg)


我们可以看到，相比于之前那种拉伸以填充整个图层的效果，新的填充模式似乎更好。

#### contentsScale

contentsScale 属性定义了寄宿图的像素尺寸和视图大小的比例，默认情况下它是一个值为1.0的浮点数。该属性其实属于支持高分辨率（Retina）屏幕机制的一部分。它用来判断在绘制图层的时候应该为寄宿图创建的空间大小，和需要显示的图片的拉伸度。如果 contentsScale 设置为1.0，表示每个点1个像素绘制图片，设置为2.0，则会以每个2个像素绘制图片，这就是我们的 Retina 屏幕。

在 contentGravity 设置为非拉伸的模式下，我们来演示一下 contentsScale 的效果。

首先我们来看一下 contentsScale 默认情况下的效果：

```
// 创建子图层
CALayer* subLayer = [CALayer new];
// 布局
subLayer.frame = CGRectMake(0, 0, 150, 150);
subLayer.position = self.view.center;
// 设置颜色
subLayer.backgroundColor = UIColor.groupTableViewBackgroundColor.CGColor;
// 设置 contents
subLayer.contents = (__bridge id)([UIImage imageNamed:@"snowman"].CGImage);
// 设置填充模式
subLayer.contentsGravity = kCAGravityCenter;
// 设置 contentsScale
subLayer.contentsScale = [UIScreen mainScreen].scale;
// 将其放大，让其模糊来演示contentsScale
subLayer.transform = CATransform3DMakeScale(3, 3, 0);
// 添加到视图关联图层上
[self.view.layer addSublayer:subLayer];
```
![效果图](https://ws3.sinaimg.cn/large/006tNbRwgy1fxdleaw9mgj30r60f2mxk.jpg)

我们发现，图片边缘部分变得模糊不清，这由于拉伸一种因素在转换的时候丢失了。我们可以手动设置 contentsScale 的值。

```
// 创建子图层
CALayer* subLayer = [CALayer new];
// 布局
subLayer.frame = CGRectMake(0, 0, 150, 150);
subLayer.position = self.view.center;
// 设置颜色
subLayer.backgroundColor = UIColor.groupTableViewBackgroundColor.CGColor;
// 设置 contents
subLayer.contents = (__bridge id)([UIImage imageNamed:@"snowman"].CGImage);
// 设置填充模式
subLayer.contentsGravity = kCAGravityCenter;
// 设置 contentsScale
subLayer.contentsScale = 2;  // 一般设置为 [UIScreen mainScreen].scale，可根据设备自动调整
// 将其放大，让其模糊来演示contentsScale
subLayer.transform = CATransform3DMakeScale(3, 3, 0);
// 添加到视图关联图层上
[self.view.layer addSublayer:subLayer];
```

![效果图](https://ws3.sinaimg.cn/large/006tNbRwgy1fxdlkoiwhzj30qu0fejrg.jpg)

这样我们的图片在 Retina 设备上显示正常。

另外，在我们使用 CATextLayer 时，我们会发现文字变成像素形式，这个时候就需要你将 contentsScale 设置为比较合适的值。

#### maskToBounds

maskToBounds 可以切除超出图层的部分，UIView中类似的有 clipsToBounds。

#### contentsRect

contentsRect 可以让我们选择图片的一个子域。它是采用单位坐标来指定区域的，默认情况下， contentsRect 是 {0, 0, 1, 1}，即显示整个寄宿图，如果我们指定小一点的区域时，图片就会被裁减显示。

![显示小区域](https://ws2.sinaimg.cn/large/006tNbRwgy1fxdm0v4bduj30u60i4gm5.jpg)

这样我们来裁剪左上角的区域作为填充图，代码修改如下：

```
// 创建子图层
CALayer* subLayer = [CALayer new];
// 布局
subLayer.frame = CGRectMake(0, 0, 150, 150);
subLayer.position = self.view.center;
// 设置颜色
subLayer.backgroundColor = UIColor.groupTableViewBackgroundColor.CGColor;
// 设置 contents
subLayer.contents = (__bridge id)([UIImage imageNamed:@"snowman"].CGImage);
// 设置 contentsRect
subLayer.contentsRect = CGRectMake(0, 0, 0.5, 0.5);
// 添加到视图关联图层上
[self.view.layer addSublayer:subLayer];
```

![效果图](https://ws1.sinaimg.cn/large/006tNbRwgy1fxdm5c084rj30ru0eyglh.jpg)

另外，我们利用 contentsRect 的特性一次载入拼合图，通过裁剪之后填充到不同图层中。这样做的好处是节省内存的使用、缩短载入时间、提高渲染性能等等。就像下面的情况：

![拼合图](https://ws4.sinaimg.cn/large/006tNbRwgy1fxdm9j3sl7j30xo0h4wf2.jpg)

经过裁剪之后，填充到不同的图层：

![裁剪图层](https://ws2.sinaimg.cn/large/006tNbRwgy1fxdmap6xakj30vy0hg3zi.jpg)

#### contentsCenter

有时候，我们需要将图片进行局部拉伸，例如在社交软件中，我们需要根据消息的文字信息将控件拉伸到合适的大小，如果该文本控件设置了背景图片，可能因为拉伸的原因产生了差异效果，这不是我们想看到的。contentsCenter 是一个 CGRect，它定义了一个固定的边框和一个在图层上可拉伸的区域。

默认情况下，contentsCenter 是{0, 0, 1, 1}。如果我们设置为{0.25, 0.25, 0.5, 0.5}，拉伸的效果就如下面：

![{0.25, 0.25, 0.5, 0.5}效果](https://ws3.sinaimg.cn/large/006tNbRwgy1fxdmkvsdg6j30ys0eeq3m.jpg)

这样当被拉伸之后，效果如下：

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxdmox3vtij30x00g0aah.jpg)

在 Interface Builder 中，可以在下图中控制 contentsCenter 属性

![Interface Builder](https://ws1.sinaimg.cn/large/006tNbRwgy1fxdmrigpvmj30e60limxj.jpg)

#### 绘制图形

除了给图层填充寄宿图，我们可以直接使用图层绘制图形。在 UIView 中，我们可以重写 `drawRect:` 来绘制图形（开发者可以调用setNeedsDisplay方法触发)。实质上该方法封装了 CALayer 的绘制方法。

在 CALayer 中，有一个 delegate 属性，你可以在代理方法中完成 CALayer 的绘制。你可以调用 `-display` 触发代理方法。

```
- (void)displayLayer:(CALayerCALayer *)layer;
```
该方法是默认的代理方法。你可以在这里设置 contents 属性。

如果不实现该代理方法，系统会调用下面的方法：

```
- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;
```

该方法传递了一个绘制的图形上下文环境，你可以使用它完成图层的绘制工作。

```
- (void)viewDidLoad {
    [super viewDidLoad];
    // 创建子图层
    CALayer* subLayer = [CALayer new];
    // 布局
    subLayer.frame = CGRectMake(0, 0, 150, 150);
    subLayer.position = self.view.center;
    // 设置颜色
    subLayer.backgroundColor = UIColor.groupTableViewBackgroundColor.CGColor;
    // 设置代理
    subLayer.delegate = self;
    // 添加到视图关联图层上
    [self.view.layer addSublayer:subLayer];
    // 触发 CALayer 的绘制
    [subLayer display];
}

// 完成绘制的代理方法
-(void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx{
    CGContextSetLineWidth(ctx, 10.0f);
    CGContextSetStrokeColorWithColor(ctx, [UIColor redColor].CGColor);
    CGContextStrokeEllipseInRect(ctx, layer.bounds);
}
```

![使用代理完成图形绘制](https://ws2.sinaimg.cn/large/006tNbRwgy1fxdnnvzn4nj30r40f8glj.jpg)

需要注意的事情：

- 我们需要显示的调用 `-display` 触发重绘
- CALayerDelegate 的代理方法并没有对超出图层的绘制内容提供绘制支持，因此即使调用 masksToBounds 为 NO时，依旧会被裁减掉

一般来说，除非你创建了一个单独的图层，你几乎没有机会用到 CALayerDelegate 协议。因为当 UIView 创建了它的宿主图层时，它就会自动地把图层的 delegate 设置为它自己，并提供了一个-displayLayer:的实现，你所需要做的就是实现 UIView 的 `-drawRect:` 方法。


## 图层几何学概念

## 图层效果

## 图层的变换

## 子类图层


