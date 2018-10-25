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




---

2018-10-25 新编





