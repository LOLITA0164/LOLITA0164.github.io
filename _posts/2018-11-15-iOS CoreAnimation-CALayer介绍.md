---
layout:     post
title:      iOS CALayer介绍
subtitle:   介绍有关CALayer相关的基础知识及相关子类的使用
date:       2018-11-15-iOS 
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

#### 布局

UIView 有三个比较重要的布局属性：frame，bounds 和 center，CALayer 对应地叫做 frame，bounds 和 position。为了能清楚区分，图层用了“position”，视图用了“center”，但是他们都代表同样的值。
frame 代表了图层的外部坐标（也就是在父图层上占据的空间），bounds 是内部坐标（{0, 0}通常是图层的左上角），center 和 position 都代表了相对于父图层 anchorPoint 所在的位置。

![UIView 和 CALayer 的坐标系](https://ws4.sinaimg.cn/large/006tNbRwgy1fxdqa445nej312w0qrgmh.jpg)

当操纵视图的 frame，实际上是在改变位于视图下方 CALayer 的frame，不能够独立于图层之外改变视图的 frame。

对于视图或者图层来说，frame 并不是一个非常清晰的属性，它其实是一个虚拟属性，是根据 bounds，position 和 transform 计算而来，所以当其中任何一个值发生改变，frame都会变化。相反，改变frame的值同样会影响到他们当中的值。

当对图层做变换的时候，比如旋转或者缩放，frame 实际上代表了覆盖在图层旋转之后的整个轴对齐的矩形区域，也就是说 frame 的宽高可能和 bounds 的宽高不再一致了。

```
CGAffineTransform transform = CGAffineTransformMakeRotation(M_PI_4);
view.transform = transform;
```

![将视图旋转45度后的布局](https://ws1.sinaimg.cn/large/006tNbRwgy1fxdqc4pp45j314j0s4wg0.jpg)

#### 锚点 

anchorPoint 可以看作是移动图层的把柄，默认情况下位于图层的中点，当你旋转视图时，都是围绕 anchorPoint 来进行旋转的，及在视图中心点做旋转。你可以通过移动 anchorPoint 来改变 frame 以及 旋转的中心点，比如你将它置于图层 frame 的左上角，这时图层会向右下角的 position 方向移动，此时虽然依旧是围绕 anchorPoint 来进行旋转，但是已经是视图的左上角了。

![改变anchorPoint的效果](https://ws4.sinaimg.cn/large/006tNbRwgy1fxe9qt206lj30u00x8404.jpg)

#### 坐标系

和视图一样，图层在图层树当中也是相对于父图层按层级关系放置，如果父图层发生了移动，它的所有子图层也会跟着移动。但是有时候你需要知道一个图层的绝对位置，或者是相对于另一个图层的位置，而不是它当前父图层的位置。

CALayer给不同坐标系之间的图层转换提供了一些工具类方法：

```
- (CGPoint)convertPoint:(CGPoint)point fromLayer:(CALayer *)layer;
- (CGPoint)convertPoint:(CGPoint)point toLayer:(CALayer *)layer;
- (CGRect)convertRect:(CGRect)rect fromLayer:(CALayer *)layer;
- (CGRect)convertRect:(CGRect)rect toLayer:(CALayer *)layer;
```

**Z坐标轴**

和 UIView 严格的二维坐标系不同，CALayer存在于一个三维空间当中。除了我们已经讨论过的 position 和 anchorPoint 属性之外， CALayer 还有另外两个属性，zPosition 和 anchorPointZ ，二者都是在Z轴上描述图层位置的浮点类型。

zPosition 最实用的功能就是改变图层的显示顺序了。通过增加图层的zPosition ，就可以把图层向相机方向前置，于是它就在小于它的zPosition 值的图层的前面。

和 UIView 添加视图一样，先添加视图数上的会被后面的视图覆盖住，图层也是同样的道理，后绘制的图层将会遮盖住之前的图层。

```
CALayer* subLayer = [CALayer new];
subLayer.frame = CGRectMake(0, 0, 150, 150);
subLayer.position = self.view.layer.position;
subLayer.backgroundColor = UIColor.redColor.CGColor;
// 先 添加到视图关联图层上
[self.view.layer addSublayer:subLayer];
CALayer* subLayer2 = [CALayer new];
subLayer2.frame = CGRectMake(0, 0, 150, 150);
subLayer2.position = CGPointMake(self.view.layer.position.x+50, self.view.layer.position.y+50);
subLayer2.backgroundColor = UIColor.blueColor.CGColor;
// 后
[self.view.layer addSublayer:subLayer2];
```

![后绘制的图层将覆盖之前的图层](https://ws3.sinaimg.cn/large/006tNbRwgy1fxegbzhtonj30qs0esa9v.jpg)

不过，类似 UIView，你可以使用一下方法调整视图的层级关系：

```
- (void)insertSublayer:(CALayer *)layer atIndex:(unsigned)idx;
- (void)insertSublayer:(CALayer *)layer below:(nullable CALayer *)sibling;
- (void)insertSublayer:(CALayer *)layer above:(nullable CALayer *)sibling;
```

当然，我们为了显示 zPosition 的作用，我们不调用上述的方法，我们来修改一下图层的 zPosition 值，其他的代码不做任何变化：

```
subLayer.zPosition = 1.0;
```

![设置zPosition后的图层关系](https://ws2.sinaimg.cn/large/006tNbRwgy1fxegfsune2j30qu0ewdfn.jpg)

#### Hit Testing

如前面所有，CALayer 和 UIView 最大的区别就是，CALayer 并不关心任何响应链事件，所以不能直接处理触摸事件或者手势，但是在实际开发过程中，我们使用了图层绘制图形并且需要处理用户的触摸事件，那该怎么办呢？好在 CALayer 有一系列的方法帮你处理触摸事件：`-containsPoint:` 和 `-hitTest:`。

`-containsPoint:`

该方法可以判断一个点是否在图层的 frame 范围内，如果在就返回 YES，反之为 NO。

判断测试的核心代码如下：

```
-(void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    CGPoint point = [[touches anyObject] locationInView:self.view];
    // 将落点转换到红色背景的图层的层级上，以判断是否被包含
    point = [self.subLayer convertPoint:point fromLayer:self.view.layer];
    if ([self.subLayer containsPoint:point]) {
        [[[UIAlertView alloc] initWithTitle:@"在红色图层中"
                                    message:nil
                                   delegate:nil
                          cancelButtonTitle:@"OK"
                          otherButtonTitles:nil] show];
    }
    else{
        [[[UIAlertView alloc] initWithTitle:@"未在红色图层中"
                                    message:nil
                                   delegate:nil
                          cancelButtonTitle:@"OK"
                          otherButtonTitles:nil] show];
    }
}
```

![触摸测试结果](https://ws2.sinaimg.cn/large/006tNbRwgy1fxehop4fv7j30qw0eu3yh.jpg)

`-hitTest:`

`-hitTest:` 同样接受一个点，但是它返回的是包含这个点的图层，而不是 BOOL 类型。如果这个点在被检测的父类图层之外，则返回 nil。

我们修改后的代码如下：

```
-(void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    CGPoint point = [[touches anyObject] locationInView:self.view];
    // 将落点转换到红色背景的图层上，以判断是否被包含
    point = [self.subLayer convertPoint:point fromLayer:self.view.layer];
    CALayer *layer = [self.view.layer hitTest:point];
    if (layer == self.subLayer) {
        [[[UIAlertView alloc] initWithTitle:@"在红色图层中"
                                          message:nil
                                         delegate:nil
                                cancelButtonTitle:@"OK"
                                otherButtonTitles:nil] show];
    }else{

        [[[UIAlertView alloc] initWithTitle:@"未在红色图层中"
                                    message:nil
                                   delegate:nil
                          cancelButtonTitle:@"OK"
                          otherButtonTitles:nil] show];
    }
}
```

![触摸测试结果](https://ws3.sinaimg.cn/large/006tNbRwgy1fxehuz2zl6j30qq0f03yh.jpg)


## 特殊效果

在我们开发过程中，想必经常遇到过设置视图的圆角，边框，阴影等特殊效果吧，实际上这些都是都过设置 CALayer 来达到效果的。如果大家都比较熟悉，就可以跳过这一节。

#### 圆角和裁剪

conrnerRadius 和 masksToBounds

CALayer 有一个叫做 conrnerRadius 的属性控制着图层角的曲率。它是一个浮点数，默认为0（为0的时候就是直角），但是你可以把它设置成任意值。默认情况下，这个曲率值只影响背景颜色而不影响背景图片或是子图层。不过，如果把 masksToBounds 设置成 YES 的话，图层里面的所有东西都会被截取。

示例：

```
// 设置圆角
self.bgLayer.cornerRadius = 20;

// 设置圆角和裁剪
self.bgLayer2.cornerRadius = 20;
self.bgLayer2.masksToBounds = YES;
```

![圆角和裁剪](https://ws2.sinaimg.cn/large/006tNbRwgy1fxeiv4weipj30rg0fajr8.jpg)

#### 边框和边框颜色

CALayer 另外两个非常有用属性就是 borderWidth 和 borderColor。二者共同定义了图层边的绘制样式。

我们修改代码如下：

```
// 设置圆角
self.bgLayer.cornerRadius = 20;
// 设置边框和边框颜色
self.bgLayer.borderWidth = 5;
self.bgLayer.borderColor = UIColor.brownColor.CGColor;

// 设置圆角和裁剪
self.bgLayer2.cornerRadius = 20;
self.bgLayer2.masksToBounds = YES;
// 设置边框和边框颜色
self.bgLayer2.borderWidth = 5;
self.bgLayer2.borderColor = UIColor.brownColor.CGColor;
```

![添加边框和边框颜色](https://ws3.sinaimg.cn/large/006tNbRwgy1fxej0e0dr8j30r20fk745.jpg)

#### 阴影

**shadowOpacity、shadowColor、shadowOffset 和 shadowRadius**

CALayer 中的常用到的四个属性 shadowOpacity、shadowColor、shadowOffset 和 shadowRadius。其中：

- shadowOpacity 控制图层阴影的透明度
- shadowColor 阴影的颜色
- shadowOffset 阴影的方向和距离
- shadowRadius 阴影的的模糊度

那么我们来给上个例子中的图层添加上阴影部分的效果：

```
self.bgLayer.shadowOpacity = 1;
self.bgLayer.shadowColor = UIColor.brownColor.CGColor;
self.bgLayer.shadowOffset = CGSizeMake(0, -3);
self.bgLayer.shadowRadius = 10;
```

![阴影效果](https://ws1.sinaimg.cn/large/006tNbRwgy1fxejj8nid2j30rc0eyt8s.jpg)

我们看到，左边图层被我们设置阴影了效果。

需要说明的是，shadowOffset 是一个 CGSize，宽度控制阴影横向位移，高度控制纵向位移，默认值是{0.-3}，即阴影相对Y轴向上位移3个点。shadowRadius 值越大阴影越模糊，默认值为3。

**寄宿图的阴影**

另外，图层的阴影是根据内容的形状产生，而不是父图层的外形或者圆角，我们从上图就可以看出来，蓝色图层是内容部分，未裁剪的部分也有阴影效果。

我们来验证一下，将填充内容改为寄宿图：

```
self.bgLayer.shadowOpacity = 1;
self.bgLayer.shadowColor = UIColor.blackColor.CGColor;
self.bgLayer.shadowOffset = CGSizeMake(0, 3);
self.bgLayer.shadowRadius = 10;
self.bgLayer.contentsGravity = kCAGravityResizeAspect;
self.bgLayer.contents = (__bridge id)([UIImage imageNamed:@"snowman"].CGImage);
```
注意：背景填充色也是寄宿图中的一部分，这里需要注意将填充色去掉。

![阴影是根据寄宿图的轮廓来确定](https://ws1.sinaimg.cn/large/006tNbRwgy1fxekiuq1fpj30pu0eowek.jpg)

**阴影的裁剪**

回到之前的例子中，我们看到在两个图层中，左边图层被我们设置上了阴影效果，接下来我们尝试给右边的有裁剪的图层添加阴影：

```
// 左边图层
self.bgLayer.shadowOpacity = 1;
self.bgLayer.shadowColor = UIColor.brownColor.CGColor;
self.bgLayer.shadowOffset = CGSizeMake(0, -3);
self.bgLayer.shadowRadius = 10;
// 右边图层
self.bgLayer2.shadowOpacity = 1;
self.bgLayer2.shadowColor = UIColor.brownColor.CGColor;
self.bgLayer2.shadowOffset = CGSizeMake(0, -3);
self.bgLayer2.shadowRadius = 10;
```

![masksToBounds裁剪掉了阴影效果](https://ws1.sinaimg.cn/large/006tNbRwgy1fxejj8nid2j30rc0eyt8s.jpg)

我们发现，右边视图并没有阴影效果。而两个图层的唯一区别就是左边未做裁剪（即 masksToBounds 设置为 YES）。那么为什么会产生这种效果差异呢？这是由于阴影通常是在图层的边界之外，而 masksToBounds 则会将超出图层边界的部分裁剪掉，因此阴影部分实际上是被裁减掉了，所以我们看不到阴影部分。

那如果我们需要裁剪掉超出部分，也需要设置阴影该怎么办呢？这时候，我们就需要额外新增加一个 frame 一致的图层作为其阴影层，以便让用户能够看到该图层的阴影（实际上是下层的阴影），我们代码修改如下：

```
// 添加阴影层
[self.view.layer addSublayer:self.bgShadowLayer];
// 将图层添加到阴影层上
[self.bgShadowLayer addSublayer:self.bgLayer2];
// 阴影层做阴影部分的设置
self.bgShadowLayer.cornerRadius = self.bgLayer2.cornerRadius;
self.bgShadowLayer.shadowOpacity = 1;
self.bgShadowLayer.shadowColor = UIColor.brownColor.CGColor;
self.bgShadowLayer.shadowOffset = CGSizeMake(0, -3);
self.bgShadowLayer.shadowRadius = 10;
```

![masksToBounds下的阴影效果](https://ws2.sinaimg.cn/large/006tNbRwgy1fxel1q6jepj30qe0eumxc.jpg)

#### mask蒙版/遮罩

我们可以通过 masksToBounds 属性可以将图层沿边界裁剪图形，通过 cornerRadius 属性可以将图层设定一个圆角。但是如果我们希望展现的内容是一个不规则的形状，那该如何做呢？

CALayer 有一个属性 mask，这个属性本身就是一个 CALayer 类型，它的内容轮廓定义了父图层的可见部分，其他部分则会被抛弃。

![mask的效果](https://ws1.sinaimg.cn/large/006tNbRwgy1fxesuy2eqwj30xc0d83ys.jpg)

下面我们演示一下，两种 mask 的效果，一种是采用图片，一种是自定义图形。

```
@interface ViewController ()
@property (strong, nonatomic)CALayer* maskLayer1;
@property (strong, nonatomic)CAShapeLayer* maskLayer2;

@property (weak, nonatomic) IBOutlet UIImageView *imageView;
@property (weak, nonatomic) IBOutlet UIImageView *imageView1;
@property (weak, nonatomic) IBOutlet UIImageView *imageView2;
@end

- (void)viewDidLoad {
    [super viewDidLoad];
    self.imageView1.layer.mask = self.maskLayer1;
    self.imageView2.layer.mask = self.maskLayer2;
}

// 寄宿图为图片

-(CALayer *)maskLayer1{
    if (_maskLayer1==nil) {
        _maskLayer1 = [CALayer new];
        _maskLayer1.frame = self.imageView.bounds;
        UIImage* image = [UIImage imageNamed:@"clip"];
        _maskLayer1.contents = (__bridge id)image.CGImage;
    }
    return _maskLayer1;
}

// 寄宿图为自定义图形

-(CAShapeLayer *)maskLayer2{
    if (_maskLayer2==nil) {
        _maskLayer2 = [CAShapeLayer new];
        _maskLayer2.frame = self.imageView.bounds;
        UIBezierPath* path = [UIBezierPath bezierPathWithOvalInRect:_maskLayer1.bounds];
        _maskLayer2.path = path.CGPath;
    }
    return _maskLayer2;
}
```

![mask的效果](https://ws4.sinaimg.cn/large/006tNbRwgy1fxeszf9ohdj30qe0f43zi.jpg)

CALayer 蒙板图层真正厉害的地方在于蒙板图不局限于静态图。任何有图层构成的都可以作为 mask 属性，这意味着你的蒙板可以通过代码甚至是动画实时生成。

如果我们将上面的 mask 蒙板图层加上动画：

```
- (void)viewDidLoad {
    [super viewDidLoad];

    self.imageView1.layer.mask = self.maskLayer1;
    self.imageView2.layer.mask = self.maskLayer2;
    
    
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.y"];
    animation.duration = 1.0f;
    animation.toValue = @(M_PI);
    animation.autoreverses = YES;
    animation.repeatCount = HUGE_VALF;
    [self.maskLayer1 addAnimation:animation forKey:@"animation"];
    [self.maskLayer2 addAnimation:animation forKey:@"animation"];
}
```

![mask图层动画](https://ws3.sinaimg.cn/large/006tNbRwgy1fxet5f45ymg30ov0dwni7.gif)

你可以利用 mask 属性做很多有趣的事情，例如你可以将文字（CATextLayer，或者UILabel.layer）加到彩色图层上，从而产生渐变效果的文字。

![渐变文字](https://ws2.sinaimg.cn/large/006tNbRwgy1fxeuvshhqfj304s018742.jpg)

## 图层的变换

在 UIView 中，有一个 transform 属性，它是一个 CGAffineTransform 类型，可以用来对视图做二维空间做旋转、缩放和平移的变换。 

实际上，UIView 的 transform 只是封装了内部图层的变换，只不过 CALayer 做二维空间变换的属性叫做 affineTransform，而其属性 transform 是一个 CATransform3D 类型，可以将图层在三维空间变换效果。

关于变换可以参考之前的一篇文章：[iOS transform变换
](http://luoliang.online/2017/06/23/iOS-transform%E5%8F%98%E6%8D%A2/)，文章中介绍的 transform 是在 UIView 的基础上介绍的，不过没关系，原理是一样的，将 UIView 的 transform 改为 CALayer 的 affineTransform 属性即可。

## 子类图层

#### CAShapeLayer

#### CATextLayer

#### CAGradientLayer

#### CAReplicatorLayer

#### CATransformLayer

#### CAScrollLayer

#### CATiledLayer

#### CAEmitterLayer

#### CAEAGLLayer

#### AVPlayerLayer



