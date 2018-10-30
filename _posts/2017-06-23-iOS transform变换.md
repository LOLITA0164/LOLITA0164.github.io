---
layout:     post
title:      iOS transform变换
subtitle:   仿射变换和3D变换
date:       2017-06-23
author:     LOLITA0164
header-img: img/post-bg-transform.jpg
catalog: true
tags:
    - iOS
    - 知识点补充
---

## 仿射变换（CGAffineTransform）

CGAffineTransform可以使控件的产生移动、缩放、旋转效果,其坐标系统采用的是二维坐标系,坐标原点为屏幕的左上角，向右为x轴正方向,向下为y轴正方向。

#### 移动控件

`CGAffineTransformMakeTranslation` 实现以初始位置为基准，在x轴方向上平移x单位，在y轴方向上平移y单位。

示例：

```
// 使用：将图片左(100px)下(150px)方向移动
CGAffineTransform transform = CGAffineTransformMakeTranslation(-100, 150);
self.imageView.transform = transform;
```
`CGAffineTransformTranslate` 在已有的transform基础上，增加 移动 效果

示例：

```
// 格式  
CGAffineTransformTranslate(CGAffineTransform t,
  CGFloat tx, CGFloat ty)
// 使用
self.imageView.transform = CGAffineTransformTranslate(self.imageView.transform, -50, 150); 
```

#### 缩放控件

`CGAffineTransformMakeScale` 实现以初始位置为基准，在x轴方向上缩放x倍,在y轴方向上缩放y倍。

示例：

```
// 使用：将图片放大2倍
self.imageView.transform = CGAffineTransformMakeScale(2, 2);
```

`CGAffineTransformScale` 在已有的transform基础上，增加 缩放 效果

示例：

```
// 使用：宽度缩小一倍，高度拉伸1.5倍
self.imageView.transform = CGAffineTransformScale(self.imageView.transform, 0.5 1.5); 
```

#### 旋转控件

`CGAffineTransformMakeRotation` 实现以初始位置为基准，将坐标系统旋转angle弧度(弧度=π/180×角度，M_PI弧度代表180角度)。

示例：

```
self.imageView.transform = CGAffineTransformMakeRotation(M_PI);
```

`CGAffineTransformRotate` 在已有的transform基础上，增加 旋转 效果

示例：

```
self.imageView.transform = CGAffineTransformRotate(self.imageView.transform,
  M_PI/2.0);
```

#### 结合变换效果

`CGAffineTransformConcat` 结合两种变换

示例：

```
//定义两种ransform
CGAffineTransform transform_A = CGAffineTransformMakeTranslation(0, 200);
CGAffineTransform transform_B = CGAffineTransformMakeScale(0.2, 0.2);
transform = CGAffineTransformConcat(transform_B, transform_A);
```

#### 反转变换效果 

`CGAffineTransformInvert` 可以实现于transform相反的效果，比如放大3倍效果则缩小为1/3，向x轴正方向平移100px效果则为向负方向平移100px。

示例：

```
CGAffineTransform transform = CGAffineTransformMakeScale(3, 3);  
//相反  缩小至1/3                
transform = CGAffineTransformInvert(transform);
self.imageView.transform ＝ transform;
```

#### 最初 transform

控件的 transform 属性默认值为 CGAffineTransformIdentity ,可以在形变之后设置该值以还原到最初状态。

示例：

```
self.imageView.transform = CGAffineTransformIdentity;
```

#### 判断 transform

`CGAffineTransformIsIdentity` 可以判断view.transform当前状态是否是最初状态：

```
bool CGAffineTransformIsIdentity(CGAffineTransform t)
```

`CGAffineTransformEqualToTransform` 可以判断两种transform是否是一样的： 

```
bool CGAffineTransformEqualToTransform(CGAffineTransform t1, CGAffineTransform t2) 
```

#### 仿射变换转换point，size，rect

`CGPointApplyAffineTransform` 可以使用一种 transform 来得到转换后的point

`CGSizeApplyAffineTransform ` 可以使用一种 transform 来得到转换后的size

`CGRectApplyAffineTransform ` 可以使用一种 transform 来得到转换后的rect

示例：

转换 point，其他类似的操作

```
CGPoint point =  CGPointMake(123, 222);
CGPoint pointNew =  CGPointApplyAffineTransform(point, CGAffineTransformMakeTranslation(77, 28));
```

#### CGAffineTransform原理

CGAffineTransform 形变是通过”仿射变换矩阵”来控制的,其中平移是矩阵相加,旋转与缩放则是矩阵相乘，CGAffineTransform 形变就是把二维形变使用一个三维矩阵来表示，系统提供了 CGAffineTransformMake 结构体来控制形变。

```
CGAffineTransformMake(CGFloat a, CGFloat b, CGFloat c, CGFloat d, CGFloat tx, CGFloat ty)
```

该三维变换矩阵如下： 

![变换矩阵](https://ws4.sinaimg.cn/large/006tNbRwgy1fwk9mjfzblj304j04g3yh.jpg)

通过变换矩阵左乘向量,将空间中的一个点集从一个坐标系变换到另一个坐标系中,计算方式如下 :

![矩阵相乘](https://ws1.sinaimg.cn/large/006tNbRwgy1fwk9ngfyfrj30ew04gmxb.jpg)

![计算结果](https://ws4.sinaimg.cn/large/006tNbRwgy1fwk9nupfnrj306902pjre.jpg)

由此可知, 

tx：用来控制在x轴方向上的平移
 
ty：用来控制在y轴方向上的平移 

a：用来控制在x轴方向上的缩放 

d：用来控制在y轴方向上的缩放 

abcd：共同控制旋转

所以以下写法都是等同的

- 移动：[ 1 0 0 1 tx ty ]

```
CGAffineTransformMakeTranslation(100, 100);
CGAffineTransformMake(1, 0, 0, 1, 100, 100);
```

- 缩放：[ sx 0 0 sy 0 0 ]

```
CGAffineTransformMakeScale(2, 0.5);
CGAffineTransformMake(2, 0, 0, 0.5, 0, 0);
```

- 旋转：[ cos(angle) sin(angle) -sin(angle) cos(angle) 0 0 ]

```
CGAffineTransformMakeRotation(M_PI*0.5);
CGAffineTransformMake(cos(M_PI * 0.5), sin(M_PI * 0.5), -sin(M_PI * 0.5), cos(M_PI * 0.5), 0, 0);
```

- 最初：[ 1 0 0 1 0 0 ]

```
CGAffineTransformIdentity;
CGAffineTransformMake(1, 0, 0, 1, 0, 0);

```

#### 仿射变换的应用

- 结合UIView动画使用

```
[UIView animateWithDuration:1.0 animations:^{
    //缩放
    CGAffineTransform transform = CGAffineTransformMakeScale(2, 2);           
    ws.imageView.transform = transform;
} completion:^(BOOL finished) {
    [UIView animateWithDuration:1.0 animations:^{
        //回到最初
        ws.imageView.transform =  CGAffineTransformIdentity;              
    } completion:nil]; 
}];
```

- 结合手势使用

```
//点击手势
UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tapAction:)];
tap.numberOfTapsRequired = 2;
[self.testView addGestureRecognizer:tap];
//拖拽手势
UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(panAction:)];
[self.testView addGestureRecognizer:pan];

#pragma mark - 点击手势
-(void)tapAction:(UITapGestureRecognizer *)tap{
    if (CGAffineTransformIsIdentity(self.testView.transform)) {
        [UIView animateWithDuration:0.5 animations:^{
            self.testView.transform = CGAffineTransformScale(self.testView.transform, 1.3, 2);
        }];
    }
    else{
        [UIView animateWithDuration:0.5 animations:^{
            self.testView.transform = CGAffineTransformIdentity;
        }];
    }
}
#pragma mark - 拖拽手势
-(void)panAction:(UIPanGestureRecognizer *)pan{
    //获取手势位置
    CGPoint position = [pan translationInView:self.testView];
    //通过 CGAffineTransformTranslate 获取 新的transform
    self.testView.transform = CGAffineTransformTranslate(self.testView.transform, position.x, position.y);
    //将增加置为 0
    [pan setTranslation:CGPointZero inView:self.testView];
}
```

效果图：

![UIView动画](https://ws3.sinaimg.cn/large/006tNbRwgy1fwk9s1f7deg308w0eeq9w.gif)

![手势](https://ws1.sinaimg.cn/large/006tNbRwgy1fwk9svut51g308w0dw0td.gif)

![加载动画](https://ws4.sinaimg.cn/large/006tNbRwgy1fwk9tem38lg308w090tac.gif)


## 3D变换

CGAffineTransform 是 Core Graphics 是 2D 平面上的坐标变换，而 iOS 开发中，会用到很多 3D 变换，这就是 Core Animation 提供的 CATransform3D，该 API 大部分都和 2D 情况类似，就不多做说明，但是这里需要特别的提议下 透视投影 这个概念，以及 m34 这个值的来历。

#### 坐标系统

iOS 设备上，是按照左手系的三维空间，即正面面对设备屏幕，坐标原点从屏幕左上方起，x轴指向右方，y轴指向下方，z轴为屏幕指向用户，就如下图：

![坐标系](https://ws3.sinaimg.cn/large/006tNbRwgy1fwq7m5xphmj30k00auwf7.jpg)

#### 变换矩阵

```
typedef struct CATransform3D {
  CGFloat m11, m12, m13, m14;
  CGFloat m21, m22, m23, m24;
  CGFloat m31, m32, m33, m34;
  CGFloat m41, m42, m43, m44;
} CATransform3D;
```

三维坐标 (x, y, z) 经过矩阵乘法等到的结果 (x', y', z') 如下：

![矩阵相乘](https://ws2.sinaimg.cn/large/006tNbRwgy1fwq7iu8db9j30an02sjri.jpg)

![结果](https://ws2.sinaimg.cn/large/006tNbRwgy1fwq7j9g82ej30eq013mxa.jpg)

#### 初始

![初始矩阵](https://ws4.sinaimg.cn/large/006tNbRwgy1fwq8y2cv7aj30gu09wdgj.jpg)


#### 平移

类似二维空间的平移，变换矩阵第四行的m41,m42,m43对应的就是x、y、z的平移量，因此矩阵变换很简单，比如将 (x, y) 向量平移到 (x+a, y+b, z+c) ：

平移矩阵：

![平移](https://ws1.sinaimg.cn/large/006tNbRwgy1fwq7qm65waj30dm02sgln.jpg)

对应的 API：

```
CATransform3D CATransform3DMakeTranslation(CGFloat tx, CGFloat ty, CGFloat tz);

CATransform3D CATransform3DTranslate (CATransform3D t, CGFloat tx,
    CGFloat ty, CGFloat tz)
```

#### 缩放

同二维空间的缩放，我们需要对向量坐标乘以系数，那么构造出来一个对角矩阵即可

缩放矩阵：

![缩放](https://ws2.sinaimg.cn/large/006tNbRwgy1fwq7sehdokj30bu02s0sr.jpg)

对应的构造API：

```
CATransform3D CATransform3DMakeScale(CGFloat sx, CGFloat sy, CGFloat sz);

CATransform3D CATransform3DScale (CATransform3D t, CGFloat sx,
    CGFloat sy, CGFloat sz)

```

#### 旋转
 
参考二维的旋转，在二维的旋转中，它是以自身的 anchorPoint 为原点，通过顺时针或者逆时针的旋转。但是在三维坐标系中，旋转就变得复杂了很多，因为向量不仅仅是在 XoY 的平面上，它可以围绕三维空间任意轴旋转，但是，为了简化，我们一般只研究三个坐标轴，即 x、y、z 轴旋转。其中，对于绕 z 轴的旋转，可以看作是等价于二维的旋转。

在二维坐标系中的旋转：

![旋转](https://ws3.sinaimg.cn/large/006tNbRwgy1fwq8qu88azj314c19mn3g.jpg)

该推导同样适用于绕 z 轴旋转的情况

![三维绕z轴旋转](https://ws3.sinaimg.cn/large/006tNbRwgy1fwq8t631lej3058024aa1.jpg)

绕x轴的旋转矩阵（固定x，从y旋转到z，即用y替换x，z替换y，x替换z）：

![绕x轴旋转](https://ws4.sinaimg.cn/large/006tNbRwgy1fwq8u30xe1j30j102s3yn.jpg)

绕y轴的旋转矩阵（固定y，从z旋转到x，即用z替换x，x替换y，y替换z）：


![绕y轴旋转](https://ws1.sinaimg.cn/large/006tNbRwgy1fwq8ukiq9uj30j302swem.jpg)

对应API：

```
CATransform3D CATransform3DMakeRotation(CGFloat radians, CGFloat x, CGFloat y, CGFloat z);

CATransform3D CATransform3DRotate (CATransform3D t, CGFloat angle,
    CGFloat x, CGFloat y, CGFloat z)
```

这个API中，radians是弧度不用说，x,y,z分别介于[-1,1]之间，表示一个任意的单位向量，绕这个单位向量的正方向旋转（[x,y,z]的长度是1，比如设置[1,0,0]就指的是绕x轴正方向旋转对应弧度值）

#### 总览

![总览](https://ws3.sinaimg.cn/large/006tNbRwgy1fwq9410c2yj30xo11cwhe.jpg)


#### 透视投影

在实际应用时，如果你直接使用旋转，视图看起来仅仅是被压扁的感觉，这是为什么呢？原因很简单，假如我们将视图绕着 y 轴做旋转，三维空间中，视图确实被旋转了，但是在显示时，我们看到是将旋转后的视图正投影到了 XoY 平面上，这种情况下，我们的视图看起来仅仅是被压扁的感觉。

那么如果想要实现透视投影（非正投影），以达到 3D 感觉，该怎么办呢？这里就需要介绍 m34 这个值的设定了。

为了简化问题，我们现在只取被投影视图上的某一个点，找到投影都对应的点。

假定现在有个三维坐标点 (x, y, z)，需要投影到 z=0 的 XoY平面上，假如我们的视角距离该平面的距离为 d，即 (0, 0, d) ，现在对 m34 赋值，进行推导运算：

![运算](https://ws1.sinaimg.cn/large/006tNbRwgy1fwq9yc9vrqj30cd02s0sr.jpg)

此时得到的向量不为齐次，需要进行齐次化，得到真正的坐标：

![齐次运算](https://ws3.sinaimg.cn/large/006tNbRwgy1fwq9yrrbasj308k013t8m.jpg)

最后对XoY平面进行投影，则最终看到的二维向量应该为:

![结果](https://ws1.sinaimg.cn/large/006tNbRwgy1fwq9zmsi7nj308c03i747.jpg)

得到透视投影下的 x 坐标是

![x](https://ws1.sinaimg.cn/large/006tNbRwgy1fwqc43ixnvj303u02qq2s.jpg)


![](https://ws2.sinaimg.cn/large/006tNbRwgy1fwqcaxy4v4j314g17w1kz.jpg)

我们等到一个重要的结论： w=-(1/d)，即，假定焦点（就是人眼）距离原点距离为d，则m34应当填写-(1/d) 

当 m34 默认为 0 的情况下，也就是认为焦点无限远，因此看起来没有任何的 3D 感。同时，我们也知道，假如我们取 d 越大，则看起来越没有投射和 3D 感；取 d 越小，则 3D 感和失真感越强烈。一般情况下，d 取值在 500～1000 之间，3D 感越好。

示例：

```
CATransform3D transform = CATransform3DIdentity;
transform.m34 = - 1.0 / 500.0;
transform = CATransform3DRotate(transform, M_PI_4, 0, 1, 0);
self.layerView.layer.transform = transform;
```

![效果图](https://ws4.sinaimg.cn/large/006tNbRwgy1fwqch6kz3mj30k009owf5.jpg)


以上内容参考自：

[Core Animation 3D 仿射变换知识](https://zhuanlan.zhihu.com/p/23359747)

---

2018-10-25 新编





