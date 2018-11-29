---
layout:     post
title:      iOS 动画篇 - UIKit动画（二）
subtitle:   UIViewPropertyAnimator 的使用介绍
date:       2018-11-27-iOS 
author:     LOLITA0164
header-img: img/post-bg-animation.jpg
catalog: true
tags: 
    - iOS
    - 动画
---

## 简单使用篇

#### 简介

iOS10带来了很多新特性，其中有个 `UIViewPropertyAnimator` 类，光从名字上就可以看出，这是一个操作属性动画的类。实际上，这个类能够让我们对视图进行动画控制，我们除了可进行正常的运行动画，如开始、暂停、重启等操作动画，还可以将动画转换为交互式动画，任意的控制时间。

它可以对视图的可动画属性进行操作，例如frame，center，alpha 和 transform等，并且可以任意的添加多个动画块和完成块，相比于之前的 UIView 动画，它改变了我们习惯的动画流程，变得更加灵活。

#### 简单例子

改变一个视图的 center 动画：

```
// 创建动画器
UIViewPropertyAnimator* animator = [[UIViewPropertyAnimator alloc] initWithDuration:1.0
                                                                              curve:UIViewAnimationCurveEaseOut
                                                                         animations:^{
    self.contentView.center = self.view.center;
}];
// 开始动画
[animator startAnimation];
```

![移动位置的动画](https://ws3.sinaimg.cn/large/006tNbRwgy1fxnr35adqsg308w03uglk.gif)

在使用 UIViewPropertyAnimator 做动画时，需要关注下面几个点：

- 包含改变一个或多个视图属性的动画块
- 用于定义动画运行过程中的时间速率曲线
- 动画的持续时间（以秒为单位）
- 动画完成块（可选）

在上面的简单实例中，我们在1秒的时间内，改变了视图的中心位置，其中动画块即为 animations 的代码块，在此代码块中我们可以针对可动动规划进行新增改变。对于运行的动画时间速率，动画器 animator 支持 UIKit 动画中的时间速率函数，即linear、ease-in、ease-out等。

一般来说，我们所创建的动画器都是处于非活跃状态，需要手动调用`-startAnimation`将其变为活跃状态执行动画。

#### 初始化动画器

UIViewPropertyAnimator 为我们提供了多个快捷创建动画器的方法。

- 使用内置`时间速率函数`

`-initWithDuration:curve:animations:`

这种方式就是我们节例子中的使用到的创建方法，curve 参数即时间速率函数，其所支持的以下几种：

```
UIViewAnimationCurveEaseInOut //缓进缓出
UIViewAnimationCurveEaseIn //缓进
UIViewAnimationCurveEaseOut //缓出
UIViewAnimationCurveLinear //线性匀速
```

![四种内置时间速率](https://ws2.sinaimg.cn/large/006tNbRwgy1fxntp7ebyxg308w08ygmk.gif)

如果所说 UIKit 提供的速率曲线函数不能够满足你的执行动画的速率要求，你还可以通过自定义来创建自己的速度曲线。

- 使用`三次贝塞尔曲线`

`-initWithDuration:controlPoint1:controlPoint2:animations:`

三次贝塞尔曲线的起点为（0,0）且其终点为（1,1），因此两个控制点的取值范围是(0,1)。

- 使用`基于弹簧的弹性`

`-initWithDuration:dampingRatio:animations:`

`dampingRatio:`所对应的参数叫做阻尼，一般去值为（0，1）较低的阻尼值对应较小阻力和在静止之前更多更大的振荡。反之则阻力大，振荡少而小。例如你想不振荡的情况下平滑的减速动画，就可以指定值为1。

```
// 创建动画器
UIViewPropertyAnimator* animator = [[UIViewPropertyAnimator alloc] initWithDuration:1.0
                                                                       dampingRatio:0.35
                                                                         animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
}];
// 开始动画
[animator startAnimation];
```

![弹性动画](https://ws1.sinaimg.cn/large/006tNbRwgy1fxns6t8wzbg308w042jre.gif) 

- 使用`自定义时间速率对象`

`-initWithDuration:timingParameters:`

该方法需要你提供支持 `UITimingCurveProvider` 协议的对象，如果你要自定义实现此协议，必须提供所有属性的实现。

系统有两个遵循该协议的类

```
UICubicTimingParameters 
UISpringTimingParameters
```
如果你查看`UICubicTimingParameters`类时，你会发现，这个类也只是提供了支持 UIKit 内置的时间速率曲线和三次贝塞尔曲线。类似的`UISpringTimingParameters `也提供了`CASpringAnimation`中的几个物理参数。

示例：我们通过该方法实现一下和上一个方法类似的效果

```
// 弹性的时间速率
UISpringTimingParameters* parameters = [[UISpringTimingParameters alloc] initWithDampingRatio:0.35];
// 创建动画器
UIViewPropertyAnimator* animator = [[UIViewPropertyAnimator alloc] initWithDuration:1.0 timingParameters:parameters];
// 由于该创建方法没有动画块，因此需要自行追加
[animator addAnimations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
}];
// 开始动画
[animator startAnimation];
```

![弹性的时间速率](https://ws1.sinaimg.cn/large/006tNbRwgy1fxntg4kt4rg308w046jre.gif)


如果说，你受不了每次都需要主动调用`-startAnimation`方法来启动视图动画，还是习惯 UIView 的快捷使用！👌，苹果似乎注意到了这一点，为了适应开发者的习惯，除了上述几种创建动画器的方式，还有一种可以启动开启动画并能返回当前动画器的方法。

- 类方法便捷

`+ runningPropertyAnimatorWithDuration:delay:options:animations:completion:`

该方法提供了动画的几个相对比较重要的参数，如动画执行时间、延迟时间、时间速率、动画块、完成块。该方法兼容了 UIView 动画块的形式。

```
[UIViewPropertyAnimator runningPropertyAnimatorWithDuration:1.0
                                                      delay:0
                                                    options:UIViewAnimationCurveEaseOut
                                                 animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
}
                                                 completion:^(UIViewAnimatingPosition finalPosition) {}];
```

![快捷使用](https://ws4.sinaimg.cn/large/006tNbRwgy1fxnu63wgrsg308w04ct8p.gif)

#### 控制动画

UIViewPropertyAnimator 遵守了 UIViewImplicitlyAnimating 协议，而UIViewImplicitlyAnimating 协议是 UIViewAnimating 协议的子类，该类定义了如何控制动画的协议。除了上一节中使用到的`-startAnimation`方法，还有其他几个控制动画的方法。

- 开始执行动画

`-startAnimation`：方法可以启动动画或者在暂停动画后恢复动画。
`-startAnimationAfterDelay:`：和上面方法类似，不过可以指定延迟执行的时间

- 暂停动画

`-pauseAnimation`：暂停动画，当使用该方法后，动画会停留在“当前位置”，会保持当前的状态。暂停后可以使用`-startAnimation`恢复，恢复的动画会从“当前位置”继续剩余的动画，包括剩余的时间。

示例：我们执行一个2秒时长的动画，在1秒处停止，延迟1秒后恢复动画，让其继续执行。

```
UIViewPropertyAnimator* animator = [UIViewPropertyAnimator runningPropertyAnimatorWithDuration:2.0 delay:0 options:UIViewAnimationOptionCurveLinear animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
} completion:nil];
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    // 暂停当前动画
    [animator pauseAnimation];
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        // 恢复动画
        [animator startAnimation];
    });
});
```

![动画的暂停和恢复](https://ws3.sinaimg.cn/large/006tNbRwgy1fxnwz9xjd3g308w042dg3.gif)

- `-stopAnimation:`：停止动画

停止动画有多种情况，由于动画状态的机制（进阶篇会讲）的存在，当我们停止动画后，这些动画状态信息何去何从？苹果给出了两种的去处，一种时清除所有状态信息，动画器重置为初始的非活跃状态，以等待下一个动画；另外一种是保留所有状态信息，等待下一步操作。这里的 withoutFinishing 参数就是用来指明去处。

参数 withoutFinishing，表示是否应执行任何最终操作。如果值为 YES，则会清除任何动画并将动画器重置为非活跃状态，并且不会执行完成块的回调。

示例：我们执行一个2秒的动画，在一秒处停止当前动画，并且在完成块中将视图的背景色更改为红色。

```
UIViewPropertyAnimator* animator = [UIViewPropertyAnimator runningPropertyAnimatorWithDuration:2.0 delay:0 options:UIViewAnimationOptionCurveLinear animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
} completion:^(UIViewAnimatingPosition finalPosition) {
    // 动画的完成回调
    self.contentView.backgroundColor = UIColor.redColor;
}];
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    [animator stopAnimation:YES];
});
```

![withoutFinishing为YES的情况](https://ws3.sinaimg.cn/large/006tNbRwgy1fxnxl082ufg308w03wjre.gif)

运行结果发现，动画在1秒处停止了，但是并没变成红色背景，这说明，此时的动画器并不会执行完成块。

当参数为 NO 时，动画器状态为 stopped，此时通常会配合`finishAnimationAtPosition：`使用，该方法可以帮助动画器执行最终的完成块的内容，当然，这两个方法的目的是停下当前动画，让你完成此刻需要完成的内容，如其他动画，之后，你再使用`finishAnimationAtPosition：`完成动画的回调以及动画需要停止的位置。

在演示示例之前，我们来介绍一下`finishAnimationAtPosition：`。

- `-finishAnimationAtPosition:`：结束动画

该方法可以将处于 stopped 状态的动画重置为非活跃状态，并执行动画的完成块。

此方法通常配合 `-stopAnimation:` 使用，并且该方法必须在动画器状态为 stopped 状态才可以，否则会出现错误。该方法的 `UIViewAnimatingPosition` 参数有一下三种：

```
UIViewAnimatingPositionEnd //动画的终点位置
UIViewAnimatingPositionStart //动画的开头位置
UIViewAnimatingPositionCurrent //动画当前位置
```
指定 UIViewAnimatingPositionCurrent 以使视图属性与其当前值保持不变。

示例：我们继续之前的例子，这次我们配合 `-finishAnimationAtPosition:` 方法使用。

```
UIViewPropertyAnimator* animator = [UIViewPropertyAnimator runningPropertyAnimatorWithDuration:2.0 delay:0 options:UIViewAnimationOptionCurveLinear animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
} completion:^(UIViewAnimatingPosition finalPosition) {
    // 动画的完成回调
    self.contentView.backgroundColor = UIColor.redColor;
}];
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    [animator stopAnimation:NO];
    // 动画器的状态必须是stopped
    if (animator.state==UIViewAnimatingStateStopped) {
        [animator finishAnimationAtPosition:UIViewAnimatingPositionCurrent];
    }
});
```

![withoutFinishing为NO的情况](https://ws1.sinaimg.cn/large/006tNbRwgy1fxo01lu6ctg308w03amx6.gif)

我们发现，动画运行1秒后停止了，并且背景色被填充为红色，这说明`-finishAnimationAtPosition:`触发完成块，这一点和之前的例子是不同的。另外，我们看到视图停下来之后就保持在了当前位置，这是因为我们给的结束位置就是 Current。下图演示了位置的不同参数的效果。

![start、current、end三种不同位置](https://ws4.sinaimg.cn/large/006tNbRwgy1fxo0c6nbd4g308w062jrn.gif)

#### 交互式动画

**fractionComplete 属性**

UIViewPropertyAnimator 类中有一个`fractionComplete`属性，这个属性表示当前动画的完成的百分比，并且这个属性不是只读的属性，这说明我们可以精准的控制动画的整个过程。利用它，我们可以制作交互式动画。交互式动画的好处是：对于多个视图、非常复杂的视图变化加以控制变得简单。

示例：

```
@interface ViewController ()
@property (weak, nonatomic) IBOutlet UIView *blueView;
@property (weak, nonatomic) IBOutlet UIView *redView;
@property (weak, nonatomic) IBOutlet UISlider *slider;
@property (strong, nonatomic) UIViewPropertyAnimator* animator;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // 初始化动画器
    
    self.animator = [[UIViewPropertyAnimator alloc] initWithDuration:2.0 curve:UIViewAnimationCurveLinear animations:^{
        // 红色视图
        
        CGRect fram = CGRectMake(self.slider.center.x - 50/2.0, self.slider.center.y - 100, 50, 50);
        self.redView.frame = fram;
        self.redView.transform = CGAffineTransformMakeRotation(M_PI);
        self.redView.backgroundColor = UIColor.blueColor;
        // 蓝色视图
        
        self.blueView.frame = fram;
        self.blueView.transform = self.redView.transform;
        self.blueView.backgroundColor = UIColor.redColor;
    }];
    [self.slider addTarget:self action:@selector(change:) forControlEvents:UIControlEventValueChanged];
}

-(void)change:(UISlider*)slider{
    CGFloat value = slider.value;
    // 更改动画完成度
    
    self.animator.fractionComplete = value;
}
```

![控制两个视图之间的动画](https://ws4.sinaimg.cn/large/006tNbRwgy1fxo2hi1w02g308w0c0wn3.gif)

需要注意的是，在使用`fractionComplete`之前，最好调用`-pauseAnimation`暂停当前动画，此时动画处于活跃状态，但非`isRunning`。

#### 修改动画

正如前面简介中提到过，UIViewPropertyAnimator 可以修改动画，甚至是在动画处于运行状态。我们可以添加多个动画块、完成块，设置是暂停掉正在执行的动画，并且修改它的剩余时间，这让我们更加的精准的控制视图的动画行为。


- `-addAnimations:`：为视图添加动画块

我们之前在使用`自定义时间速率对象`初始化动画器时，曾经使用到过该方法，此方法可以让我们对视图的动画追加多个动画块。

示例：我们为正在运动的视图添加渐变动画

```
UIViewPropertyAnimator* animator = [UIViewPropertyAnimator runningPropertyAnimatorWithDuration:1.0 delay:0 options:UIViewAnimationCurveLinear animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
} completion:nil];
// 追加动画块
[animator addAnimations:^{
    self.contentView.backgroundColor = UIColor.redColor;
}];
```
![追加动画块](https://ws3.sinaimg.cn/large/006tNbRwgy1fxoou3118gg308w03kweg.gif)

我们追加的动画块会和其他动画共享动画器剩余的时间。

示例：延迟追加动画

为了明显的看出效果，我们给予更长的动画时间，并在运行一段时间后，追加动画。

```
UIViewPropertyAnimator* animator = [UIViewPropertyAnimator runningPropertyAnimatorWithDuration:4.0 delay:0 options:UIViewAnimationCurveLinear animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
} completion:nil];
// 追加动画块
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    [animator addAnimations:^{
        self.contentView.backgroundColor = UIColor.redColor;
    }];
});
```

![共享时间](https://ws3.sinaimg.cn/large/006tNbRwgy1fxop3y5ul0g308w036gm3.gif)

我们看到2秒后追加的渐变动画在剩余的2秒内完成了渐变效果。

当然，苹果已经给出了类似的方法，无需我们主动写延迟方法。就像下面的演示。

- `-addAnimations:delayFactor:`：延迟追加动画块

参数`delayFactor`是指时间因子，即动画的进度，取值区间为（0，1）。比如，0.5表示动画执行一半的时候执行。

示例：我们使用该方法完成之前的例子

```
UIViewPropertyAnimator* animator = [UIViewPropertyAnimator runningPropertyAnimatorWithDuration:4.0 delay:0 options:UIViewAnimationCurveLinear animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
} completion:nil];
// 延迟追加动画块
[animator addAnimations:^{
    self.contentView.backgroundColor = UIColor.redColor;
} delayFactor:0.5];
```

![延迟加载动画](https://ws3.sinaimg.cn/large/006tNbRwgy1fxop3y5ul0g308w036gm3.gif)

- `-addCompletion:`：追加完成块

既然动画块都可以追加修改，那么完成块也应该相应的有追加方法呀！

在初始化以动画器一节中，我们发现大部分都是不带有完成块回调的，苹果似乎考虑到开发过程中很少会关心动画的完成事件吧，因此为了方法的简洁性，就让其变成了可选特性，又或者这样设计会让动画变得更加的灵活，因为这样，动画的完成事件就无需紧跟在初始化方法上了。

示例：我们在动画执行完成时执行一些事情

这里为了方便看到效果，我们就直接来改变视图的颜色。

```
UIViewPropertyAnimator* animator = [[UIViewPropertyAnimator alloc] initWithDuration:1.0 curve:UIViewAnimationCurveLinear animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
}];
[animator startAnimation];
// 追加完成块
[animator addCompletion:^(UIViewAnimatingPosition finalPosition) {
    self.contentView.backgroundColor = UIColor.redColor;
}];
```

![追加完成块](https://ws4.sinaimg.cn/large/006tNbRwgy1fxopnxamjag308w02sq2w.gif)

在【控制动画】一节中，我们提到过，我们可以调用`-startAnimation`：来恢复暂停后的动画，但是这样做的话，动画的形式依旧是之前设置好的情况，它并不会发生变化。那么，如果想要暂定动画后，执行其他时间速率的动画该怎么办呢？别急，既然说了 UIViewPropertyAnimator 可以让我们任意的控制动画，必然会提供该类方法。

- `-continueAnimationWithTimingParameters:durationFactor:`：暂停后修改动画方式继续执行

该类方法只会在调用`-pauseAnimation`方法之后起到作用，此时的动画状态为，活跃但非`isRunning`。

参数`durationFactor`是时间因子，表示动画的进度。通常可以取`fractionComplete`属性。

示例：我们将匀速运行中的视图中途改为弹性运动

```
UIViewPropertyAnimator* animator = [[UIViewPropertyAnimator alloc] initWithDuration:2 curve:UIViewAnimationCurveLinear animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
}];
[animator startAnimation];
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    // 暂停动画之后
    [animator pauseAnimation];
    // 暂停动画之后，修改动画时间速率后继续动画
    UISpringTimingParameters* timing = [[UISpringTimingParameters alloc] initWithDampingRatio:0.2];
    [animator continueAnimationWithTimingParameters:timing durationFactor:animator.fractionComplete];
});
```

![中途更改动画](https://ws1.sinaimg.cn/large/006tNbRwgy1fxos16vqpag308w04s74v.gif)

上图演示了中途更改的动画情况，其中，底下的视图为参考视图，即未改变的匀速运动。

以上就是快速入门使用的所有教程了，相信经过一系列的介绍之后，你能够快速的使用新的动画方式了。

接下来的进阶篇，会讲解一些 UIViewPropertyAnimator 的一些细节部分。

## 进阶篇

#### 动画协议

其实要介绍 UIViewPropertyAnimator 类前，应该先介绍其遵循的动画协议--UIViewAnimating 和 UIViewImplicitlyAnimating 。前者是后者的父类，我们先来解释 UIViewAnimating 协议。该协议定义了操作动画的基本方法，包括启动、停止、暂停动画的能力。另外还有几个属性用于反映动画的当前状态信息。

UIViewPropertyAnimator 遵循并实现了 UIViewAnimating 协议的所有方法，因此我们可以用 UIViewPropertyAnimator 来实现动画的控制。如果你想在自定义类中也遵循此协议，最好实现所有的协议方法和属性。

**动画的状态**

UIViewPropertyAnimator 有着一套完整的状态机制。在动画器处理一组动画时，都会伴随着这一系列的动画状态。这些状态定义了动画器的行为，包括它是如何处理变化的。如果你在实现自定义动画器，必须遵循这些状态的转换并准确的更新状态属性。下图显示了发生的状态和状态转换关系。

![状态转换关系](https://ws3.sinaimg.cn/large/006tNbRwgy1fxovzi4ee1j30kq0d50sw.jpg)

`Inactive`非活跃状态是动画器的初始状态。每个新创建的动画器都会处于非活跃状态下启动。相对的，动画正常完成后会返回到非活跃状态。

当我们调用`-startAnimation `或`-pauseAnimation`方法时，此时动画器会变为`Active`活跃状态。此状态下的动画器正在运行或暂停状态。如果是动画被暂停，我们此时还可以修改动画时间速率曲线，让后让其继续运行到预期结束，结束后的动画器状态依旧是`Inactive`非活跃状态，等待我们使用一组新的动画重新配置它，以便开始新的动画。

当我们开启动画之后，调用`-stopAnimation:`方法会停止正在运行的动画，此时视图会被保留在停止的那一刻的值。此方法的参数值决定了当前的动画信息是否被擦除。如果参数 withoutFinishing 是 YES，则表示擦除当前动画的信息，动画器进入`Inactive`状态，需要注意，这种情况下，动画器是不会执行完成块的，换句话说你无法在完成块中得到动画结束信息，此时如果你需要知道动画结束的事件，你可以使用 KVO 的方法监听属性`isRunning`获得。如果参数 withoutFinishing 是 NO，则表示保留当前动画的信息，动画器进入`Stopped`状态，此时我们可以去完成其他的操作，如执行其他动画。然后我们调用方法`-finishAnimationAtPosition:`以结束此次动画，动画器顺理成章的进入到`Inactive`状态。注意，这情情况下，动画器可以顺利的执行完成块内容。

动画状态的几个枚举：

```
typedef NS_ENUM(NSInteger, UIViewAnimatingState)
{
    UIViewAnimatingStateInactive, // The animation is not executing.
    UIViewAnimatingStateActive,   // The animation is executing.
    UIViewAnimatingStateStopped,  // The animation has been stopped and has not transitioned to inactive.
} NS_ENUM_AVAILABLE_IOS(10_0) ;
```

**协议内容**

**方法**

- `-startAnimation` 开始动画

不可以在动画器调用方法`-stopAnimation:`直接结束动画后再次调用`-startAnimation`，换句话说，使用过程中，出现下面情况会出错：

![错误](https://ws2.sinaimg.cn/large/006tNbRwgy1fxoy0c0ysej31tm05qglu.jpg)

![错误](https://ws1.sinaimg.cn/large/006tNbRwgy1fxoy270v5vj30vm03m74f.jpg)

我们发现，在我们`-stopAnimation:`指定参数为 YES时，动画器状态由活跃状态`Active `转变为非活跃状态`Inactive`，此时再次调用`-startAnimation`时，系统抛出了异常。

而我们指定参数为 NO时，

![正常](https://ws1.sinaimg.cn/large/006tNbRwgy1fxoy14qs98j31pe05imxd.jpg)

此时动画器状态为`stopped`，程序并未出错。

![正常](https://ws3.sinaimg.cn/large/006tNbRwgy1fxoxp1rcssj30ii020745.jpg)

这一点和官方文档的说明并不一致，目前还不是很清楚原因。
```
It is a programmer error to call this method while the state of the animator is set to UIViewAnimatingStateStopped.
```

- `-startAnimationAfterDelay:` 延迟后开始动画

上面的开启动画一样的注意点同样适用。另外经测试发现，`-pauseAnimation`之后调用`-startAnimationAfterDelay:`会发生程序错误。

- `-pauseAnimation` 暂停动画

暂停动画后，可以使用`-startAnimation`方法重新恢复动画，另外你也可以使用协议`UIViewImplicitlyAnimating`中的`continueAnimationWithTimingParameters:durationFactor:`恢复动画。如果动画已经暂停，则再次调用`-pauseAnimation`不会执行任何操作。

经测试发现如果动画器从未启动过，直接调用`-pauseAnimation`方法，如果紧接着调用`-startAnimation`或者`continueAnimationWithTimingParameters:durationFactor:`是无法恢复动画的，之间需要大于千分之一秒的时间，就像下面的情况：

无法恢复动画的情况：

```
[self.animator pauseAnimation];
[self.animator startAnimation];
//[self.animator continueAnimationWithTimingParameters:[[UICubicTimingParameters alloc] initWithAnimationCurve:UIViewAnimationCurveLinear] durationFactor:0.5];
```

如果之前调用过开启动画，则可以恢复动画

```
[self.animator startAnimation];
...
[self.animator pauseAnimation];
[self.animator startAnimation];
//[self.animator continueAnimationWithTimingParameters:[[UICubicTimingParameters alloc] initWithAnimationCurve:UIViewAnimationCurveLinear] durationFactor:0.5];
```

又或者添加延迟

```
// 从未开启过，暂停动画
[self.animator pauseAnimation];
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.002 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    [self.animator startAnimation];
//    [self.animator continueAnimationWithTimingParameters:[[UICubicTimingParameters alloc] initWithAnimationCurve:UIViewAnimationCurveLinear] durationFactor:0.5];
});
```

至于为什么会时这种情况，官方文档并未指出，或许只有苹果自己清楚吧。

另外，官方文档指出动画器为`stopped`状态时，调用`-pauseAnimation`会出现程序错误，但并没有。

- `-stopAnimation:` 停止动画

需要注意的是，不可以在动画器状态为`stopped`的时候再调用该方法。下面的使用会发生程序错误：

![错误使用](https://ws1.sinaimg.cn/large/006tNbRwgy1fxoxz4b2h5j31to06m74p.jpg)

这一点，官方文档却并未提及。why？

- `-finishAnimationAtPosition:` 结束动画

该方法通常会和上面的停止方法相结合，用来结束当前停止下来（状态为`stopped`）动画顺利回到`Inactive`状态。经测试，该方法只认`stopped`状态，其他两个状态都会发生错误。

**属性**

- `fractionComplete`动画执行的进度

描述了当前动画的进度，可被更改，当动画处于停止时，可配合手势等实现交互式动画。

- `reversed`是否可以反转动画

关于反转动画，目前还未知如果实现反转动画。

- `state`动画器的状态

- `running`动画的运行状态，支持KVO

动画状态的变换以及停止动画和finishAnimationAtPosition，完成块的使用

针对同一属性的动画

#### 修改动画协议

UIViewImplicitlyAnimating 继承自 UIViewAnimating 协议，在后者协议的基础上又添加了一些额外的修改动画的方法。而我们使用的 UIViewPropertyAnimator 动画器就遵循了这个相对完善的协议，并实现了所有的方法。

**方法**

- `-addAnimations:` 添加动画块

使用此方法可以将新的动画块添加到自定义动画对象。新的动画会与先前的动画一起运行，并从当前时间开始并与任何原始动画同时结束。

- `-addAnimations:delayFactor:`添加延迟动画块

同上，不过会从指定的延迟开始并与任何原始动画同时结束。

参数 delayFactor：用于延迟动画开始的时间因子。该值必须介于0.0和1.0之间。将此值乘以动画剩余持续时间，作为实际延迟。例如，如果值0.5、动画器的持续时间为2.0，则延迟一秒执行动画。

- `-addCompletion:`添加动画完成块

回调动画完成的事件，你可以在该 block 中完成其他操作。

参数 withoutFinishing 有三种，表示最后动画结束的位置。

```
typedef NS_ENUM(NSInteger, UIViewAnimatingPosition) {
    UIViewAnimatingPositionEnd,
    UIViewAnimatingPositionStart,
    UIViewAnimatingPositionCurrent,
} NS_ENUM_AVAILABLE_IOS(10_0);
```

如果动画正常完成结束，位置参数为`UIViewAnimatingPositionEnd`，即最终的期望位置；

如果动画执行过程中，调用了`-stopAnimation:`，并且制定的参数为 YES，动画器则不会调用完成块；

如果动画执行过程中，调用了`-stopAnimation:`，并且制定的参数为 NO，此时需要调用`finishAnimationAtPosition：`配置结束动画，此时完成块中的位置参数由方法`finishAnimationAtPosition：`决定。

- `-continueAnimationWithTimingParameters:durationFactor:`调整暂停的动画的时间速率曲线和持续时间

参数 parameters 是指时间速率曲线，系统提供了两种，兼容了 UIKit 内置的四种时间速率曲线、三次贝塞尔曲线、弹簧式的弹性动画。

```
UICubicTimingParameters
UISpringTimingParameters
```
参数 durationFactor 是指动画原始持续时间的因子，取值为（0，1），将此值乘以动画的原始持续时间，作为新的持续时间。

```
UIViewPropertyAnimator* animator = [[UIViewPropertyAnimator alloc] initWithDuration:2 curve:UIViewAnimationCurveLinear animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
}];
[animator startAnimation];
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    [animator pauseAnimation];
    [animator continueAnimationWithTimingParameters:[[UICubicTimingParameters alloc] initWithAnimationCurve:UIViewAnimationCurveLinear] durationFactor:animator.fractionComplete];
});
```

![修改动画持续时间](https://ws4.sinaimg.cn/large/006tNbRwgy1fxp0ox2v3sg308w03y0sw.gif)

如果我们将 durationFactor 传递为当前动画器的进度值 fractionComplete ，你会发现执行的动画并没有什么变化，这是因为 fractionComplete 的值乘以原始持续时间就等于动画剩余的时间。

但是我们将值放大，比 fractionComplete 的值要大，那么动画的剩余时间就会被拉长，剩下的动画会在新的时间内完成。

示例：我们将动画1秒后，将剩余的时间缩短为0.1倍

```
UIViewPropertyAnimator* animator = [[UIViewPropertyAnimator alloc] initWithDuration:2 curve:UIViewAnimationCurveLinear animations:^{
    self.contentView.center = CGPointMake(self.view.center.x+100, self.view.center.y);
}];
[animator startAnimation];
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    [animator pauseAnimation];
    [animator continueAnimationWithTimingParameters:[[UICubicTimingParameters alloc] initWithAnimationCurve:UIViewAnimationCurveLinear] durationFactor:0.1];
});
```

![修改剩余动画的持续时间](https://ws2.sinaimg.cn/large/006tNbRwgy1fxp0ryuj6cg308w03y74b.gif)

当前就像之前介绍的，你也可以改换当前动画的时间速率曲线，或者更换为弹性动画。

#### 动画器

介绍完 UIViewPropertyAnimator 的两种协议之后，我们来看下 UIViewPropertyAnimator 中的一些其小细节。

**三次贝塞尔曲线**

在构建动画器的方法中，有一个之前我们一提而过的方法：

`-initWithDuration:controlPoint1:controlPoint2:animations:`

这个方法可以让我们自定义时间速率曲线，采用的是三次贝塞尔曲线，可能有些人并不清楚什么是三次贝塞尔曲线。三次贝塞尔曲线在绘制图形时经常出现，是有起点、终点以及两个控制点产生的曲线。文字描述比较抽象，我们来看下图：

![三次贝塞尔曲线](https://ws1.sinaimg.cn/large/006tNbRwgy1fxp10wndzbj30i80hfglo.jpg)

该曲线的起点为（0，0），其终点为（1，1）。point1 和 point2 参数是定义生成的贝塞尔曲线形状的控制点。其中，起点和控制点1的连线为曲线的切线，终点和控制点2的连线也是曲线的切线，控制点1和控制点2的连写也是曲线的切线，这样，产生的曲线就是三次贝塞尔曲线。

该曲线的斜率定义了动画的不同时间速率，斜率越大，速度越快，斜率越小速度越慢。上图显示了一个速率曲线，其中动画快速启动并快速完成，但在中间部分运行得相对较慢。

**属性**

- `duration` 只读

动画的持续时间，只有在初始化动画器时指定该值，稍后添加的动画仅在剩余的时间内运行。剩余时间由公式（1.0 - fractionComplete）* 持续时间确定。

- `delay` 只读

延迟动画时间，默认值为0。如果要为此属性设置值，启动动画时则需要使用`startAnimationAfterDelay:`方法

- `timingParameters` 只读

描述速度的曲线，和duration一样，只有在初始化 animator 指定该值。可以使用此属性稍后获取这些参数。

- `interruptible`

动画中是否可被打断。当此属性的值为 YES 时，我们可以使用`-pauseAnimation`和`-stopAnimation:`方法来中断动画并进行更改。当此属性的值为 NO 时，在调用startAnimation方法后，动画将运行至完成（并且不会中断）。

如果使用动画器来实现可中断的视图控制器转换，则此属性必须为 YES。

- `userInteractionEnabled`

动画中用户是否可交互。默认值为 YES。当此属性的值为 YES 时，触摸事件将正常传递给视图，否则在动画持续时间内会忽略用户的触摸事件。

- `manualHitTestingEnabled`

动画中点击测试的能力。默认为 NO。

- `scrubsLinearly` 

暂停的动画是否使用线性擦除或者使用指定的时间速率曲线。iOS11之后可用。

- `pausesOnCompletion` 

动画完成后是否保持活动状态。默认值为 NO。iOS11之后可用。

当此属性的值为 YES 时，动画器完成后动画后将保持`Active`状态，并且不会执行完成块。此时我们可以撤消动画。当此属性的值为 NO 时，动画完成后，动画器执行完成块，自动转换为`Inactive`状态，从而结束动画。

注：由于 YES 的情况下，动画器并且不会执行完成块，因此如果你想要知道动画的结束事件，你需要监听动画器的`running`属性。


**补充**

- 设置同一可动画属性

`-addAnimations:`方法可以让我们添加多个属性动画块，那么，如果两个或多个动画需要同时改变相同的属性会发生什么呢？苹果才用的是“后者优先”原则。即：后天加的动画效果会覆盖之前的效果。但有趣的是，这将导致卡顿，因为需要组合新旧动画。在旧动画淡出的同时会隐约看见新动画。

示例：

```
UIViewPropertyAnimator* animator = [UIViewPropertyAnimator runningPropertyAnimatorWithDuration:1.0 delay:0 options:UIViewAnimationOptionCurveLinear animations:^{
    self.contentView.transform = CGAffineTransformMakeScale(1.5, 1.5);
[animator addAnimations:^{
    self.contentView.transform = CGAffineTransformMakeScale(0.5, 0.5);
}];
```

![设置同一动画属性](https://ws4.sinaimg.cn/large/006tNbRwgy1fxp6dpkg7hg308w08c3yu.gif)

我们发现，后添加的缩小为0.5的动画效果覆盖了之前的放大为1.5的动画效果。但这似乎看不出所为卡顿的效果，那我们来看下填充背景色会发生什么。

```
UIViewPropertyAnimator* animator = [UIViewPropertyAnimator runningPropertyAnimatorWithDuration:1.0 delay:0 options:UIViewAnimationOptionCurveLinear animations:^{
    self.contentView.backgroundColor = UIColor.redColor;
} completion:nil];
[animator addAnimations:^{
    self.contentView.backgroundColor = UIColor.yellowColor;
}];
```

![设置同一动画属性](https://ws1.sinaimg.cn/large/006tNbRwgy1fxp6pdr3v2g308w05smxk.gif)

该示例中，视图的最初颜色为蓝色，在第一个动画块中，我们将其设置为红色，后又设置为黄色。在该动画运行过程中，我们发现，视图立刻被设置为红色，然后由红色渐变为黄色。因此，在多个动画块中设置同一个动画属性并不可控，我们应该尽可能的避免这种情况的出现。























