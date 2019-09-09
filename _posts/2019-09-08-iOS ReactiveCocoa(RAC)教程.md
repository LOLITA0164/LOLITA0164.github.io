---
layout:     post
title:      iOS ReactiveCocoa(RAC)教程
subtitle:   函数式响应式编程
date:       2019-09-06
author:     LOLITA0164
header-img: img/post-bg-server.jpg
catalog: true
tags: 
    - iOS 
---

[官方文档](https://www.raywenderlich.com/2493-reactivecocoa-tutorial-the-definitive-introduction-part-1-2)
[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)

##第一部分:【基础教程】

**1.0 简介**

作为iOS开发人员，我们所编写的代码几乎都是对某些事件的响应，如按钮点击、收到网络消息、属性更改（KVC键值观察）或者通过 CoreLacation 更改用户位置等等。但是，这些事件都以不同的方式进行传递，如actions、代理、KVO、回调块、通知等等多种形式，虽然灵活多样，但也同时增加了代码的复杂性。

[[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
](https://github.com/ReactiveCocoa/ReactiveCocoa) 则定义了事件的标准接口，并且可以任意的使用一组基本工具单元轻松地链接、过滤、组合他们。

**1.1 特点**

ReactiveCocoa 结合了几种编程风格：

- 函数式编程：使用更高阶的函数，即用其他函数作为参数的函数。ReactiveCocoa 中使用了大量的回调块。
- 响应式编程：重点关注数据流和变化传递

因此，ReactiveCocoa 又被广泛的描述为函数式响应编程（FRP）框架。

**2.0 演示项目**

[入门项目](https://koenig-media.raywenderlich.com/uploads/2014/01/ReactivePlayground-Starter.zip)：该项目是一个非常简单的应用程序，它需要用户提供账号密码信息，在提供了正确的凭证之后，迎接你的是一张开爱的小猫照片。

![项目结果](https://upload-images.jianshu.io/upload_images/1167313-8220299181e76d7b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 目前项目中没有涉及到任何 ReactiveCocoa 的内容，是我们编程的正常思路。

**2.1 安装 ReactiveCocoa**

关于 [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
项目的安装，您可以自行查看 GitHub 文档，按照具体演示的步骤操作即可。

**2.2 简单体验**

正如简介中所描述的那样，ReactiveCocoa 提供了一个标准接口，用于处理应用程序中发生的不同事件流。在 ReactiveCocoa 术语中被描述为**信号**，由 `RACSignal` 类表示。

首先打开应用程序的初始控制器`RWViewController.m`，并在文件顶部导入即将使用的库头文件。

```
#import <ReactiveCocoa / ReactiveCocoa.h>
```

我们先不改变任何代码的情况下，体验一下信号`RACSignal` ，在`viewDidLoad ` 中添加一下代：

```
[self.usernameTextField.rac_textSignal subscribeNext:^(id x) {
        NSLog(@"%@",x);
}];
```

ReactiveCocoa 通过分类扩展了`UITextFiled`的方法，为其添加创建信号`RACSignal` 实例的方法。`subscribeNext:`方法将`UITextFiled`变化的数据结果回调到块中。

运行应用程序，并在账号文本框中输入一些文本数据，观察控制台输出内容。

![输出结果](https://upload-images.jianshu.io/upload_images/1167313-53e3557dc2f8e49c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到，每次更改文本框中的文本数据时，回调块中的代码都会执行，并且没有target-action，也没有代理，有的仅仅是`Signal` 和结果回调。

**2.3 信号Signal**

ReactiveCocoa 中信号（由`RACSignal`）向其订阅者发送事件流。有三种类型的事件：**next**、**error**和**completed**。**信号可能在发生错误或完成的终止前发送任意数量的next事件**。在第一部分教程中，我们将只关注于`next`事件，在第二部分演示`error`和`completed`事件。

`RACSignal` 可以通过许多方法来订阅这些不同类型的事件。每个方法都需要一个或者多个Block块，当订阅的事件发生时，Block块将会被执行，您可以在Block块中获取到信号处理的结果。可以通过方法`subscribeNext:`方法添加在每个下一个事件上执行的块。

ReactiveCocoa 框架通过分类向许多标准的 UIKit 控件添加了信号，您可以向其事件添加订阅。

**2.4 添加过滤条件**

ReactiveCocoa 有着大量可以用于操作事件流的运算符。假如您只对用户长度大于三个字符的感兴趣，您可以通过`filter`运算符来实现此目的。

```
[[self.usernameTextField.rac_textSignal
  filter:^BOOL(id value) {
      return [value length] > 3;
}] subscribeNext:^(id x) {
    NSLog(@"%@",x);
}];
```
运行项目，您会发现只有当文本字段长度大于三个字符时才开始回调结果。

上述例子中创建了一个非常简单的管道，这正是 Reactive Programming 的本质，您可以根据数据流表达应用程序的功能。

我们用图形方式来描绘这些数据流：

![数据流描绘应用程序的功能](https://upload-images.jianshu.io/upload_images/1167313-c8cf72abd452502d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图中，您可以看到`rac_textSignal `信号是事件的初始来源。如果数据`filter`包含长度大于3的字符串，则允许事件通过。管道中的最后一步是`subscribeNext :`该方法中的回调用于传递结果数据。

值得提的是：`filter`操作的返回值也同样是一个`RACSignlal`,这使得信号可以继续向下传递。当然您也可以分解管道组成元素，就像下面的演示：

```
// 创建UITextField的信号对象
RACSignal* usernameSourceSignal = self.usernameTextField.rac_textSignal;
// 添加过滤
RACSignal* fileteredUsernameSignal = [usernameSourceSignal filter:^BOOL(id value) {
    return [value length] > 3;
}];
// 添加订阅回调
[fileteredUsernameSignal subscribeNext:^(id x) {
    NSLog(@"%@",x);
}];
```
因为每个操作`RACSignal`也返回一个`RACSignal`，我们称之为 `fluent interface` （我理解为面向接口式编程、链式编程，如Swift语言，SwiftUI语法）,此功能让您在构建管道时无需使用局部变量去引用每一个步骤，就像上面拆解成每一步。

**2.4 转换**

先将拆分的各种`RACSignal`重新组装成管道。

```
[[self.usernameTextField.rac_textSignal
  filter:^BOOL(id value) {
      return [value length] > 3;  //隐式转换
}] subscribeNext:^(id x) {
    NSLog(@"%@",x);
}];
```
在上述代码中，来自`id`隐式转换`NSString`不够优雅。好在传递给此块的值是确定的`NSString`类型，因此您可以更改参数类型。

```
[[self.usernameTextField.rac_textSignal
  filter:^BOOL(NSString* text) {
      return text.length > 3;
}] subscribeNext:^(id x) {
    NSLog(@"%@",x);
}];
```


**2.5 添加事件？管道的结构**

到目前为止，本教程描述了不同的事件类型，但是没有详细说明这些事件的组成规则，即管道的结构。你可以理解管道是有各种事件单元组装而来，事实上，你可以在管道上添加任何事件单元。

为了说明上面的表述，我们向管道添加另一个操作。

```
[[[self.usernameTextField.rac_textSignal
   map:^id(NSString* text) {
       return @(text.length);
   }]
  filter:^BOOL(NSNumber* length) {
      return length.integerValue > 3;
}] subscribeNext:^(id x) {
    NSLog(@"%@",x);
}];
```

运行应用程序，您将发现，控制台输出的是文本的长度而不再是内容：

![新的输出结果](https://upload-images.jianshu.io/upload_images/1167313-e3260e74071032e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

新添加的映射操作将上一个`RACSignal`传递过来的文本数据转换为NSNumber数据，并将该数据作为下一个事件发出的返回值。

用图形描述为：

![添加事件](https://upload-images.jianshu.io/upload_images/1167313-ad33458b5804f8da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图所示，`map`操作使后面的接收者都接收到NSNumber实例。您可以使用该操作将接收到的数据转换为你所需要的任何内容，当然前提它必须是一个对象，如果是基本数据类型，需要进行装箱操作，如上述中的`@(text.length)`

至此，删除之前所添加的所代码，我们利用之前所学将项目中业务重构。

**3.0 创建验证状态信号**

项目中需要监听文本框输入的内容并进行文本字段的校验。现在利用`RACSignal`来创建这些信号。

```
// 创建验证用户名的信号
RACSignal* validUsernameSignal = [self.usernameTextField.rac_textSignal map:^id(id value) {
    return @([self isValidUsername:value]);
}];
// 创建验证密码的信号
RACSignal* validPasswordSignal = [self.passwordTextField.rac_textSignal map:^id(id value) {
    return @([self isValidPassword:value]);
}];
```
上述代码使用`map`转换`rac_textSignal`事件中的文本数据转换为我们验证后的数据，由于它是布尔类型，我们通过装箱转换为`NSNumber`对象类型。

下一步再次转换这些信号，以便它们可以为文本框提供漂亮的背景颜色。

```
[[validPasswordSignal
  // 转换为颜色数据
  map:^id(NSNumber* passwordVaild) {
    return [passwordVaild boolValue] ? UIColor.clearColor : UIColor.yellowColor;
}]
 // 将颜色设置到对应的文本框上
 subscribeNext:^(UIColor* color) {
    self.passwordTextField.backgroundColor = color;
}];
```
从概念上来讲，上述代码讲信号的最终输出分配给了文本框的`backgroundColor`属性，但是从代码上很难感受出来！新运的是，ReactiveCocoa 中有一个宏，可以让你优雅地表达这一点。

```
RAC(self.usernameTextField, backgroundColor) = [validUsernameSignal map:^id(id value) {
    return [value boolValue] ? UIColor.clearColor : UIColor.yellowColor;
}];
RAC(self.passwordTextField, backgroundColor) = [validPasswordSignal map:^id(id value) {
    return [value boolValue] ? UIColor.clearColor : UIColor.yellowColor;
}];
```
宏`RAC`允许将信号的输出绑定到一个对象的属性上。它需要两个参数，第一个是某个对象，第二个是该对象属性名称。每次信号发生下一个事件时，传递的值都将分配给指定的属性。

在验证结果之前，找到`updateUIState`方法并删除前两行，清除影响验证的干扰代码：

```
self.usernameTextField.backgroundColor = self .usernameIsValid？[ UIColor clearColor]：[ UIColor yellowColor];
self.passwordTextField.backgroundColor = self .passwordIsValid？[ UIColor clearColor]：[ UIColor yellowColor];
```
运行应用程序，会发现文本框在无效时会突出显示，并在有效时清除背景色。

我们使用图来可视化当前的逻辑：

![事件传递](https://upload-images.jianshu.io/upload_images/1167313-293332a586299e96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们创建了两个管道，它们接收文本信号，并将它们`map`为有效性指示布尔值，然后使用了第二个`map`到一个`UIColor`并绑定到文本框的背景颜色属性上。

**3.1、组合信号**

在当前应用程序中，只有当用户名和密码文本字段都有有效输入时，“login”按钮才有效。

在此之前，我们都只是创建了单一的信号，`validUsernameSignal`和`validPasswordSignal`。现在，我们根据需要来组合这两个信号，以便确定何时可以启用登陆按钮。

```
RACSignal* signUpActiveSignal = [RACSignal combineLatest:@[validUsernameSignal, validPasswordSignal] reduce:^id(NSNumber* usernameValid, NSNumber* passwordValid){
    return @([usernameValid boolValue]&&[passwordValid boolValue]);
}];
```
上述代码组合`validUsernameSignal`和`validPasswordSignal`得到了一个全新的信号，每当这两个信号的其中任何一个发出新值时，`reduce`块都会被执行，它返回的值将作为组合信号的下一个值发送。

> `RACSignal`组合方法可以结合任何数量的信号，并且在`reduce`块的参数分别对应每个信号传递的值

现在，我们使用这个全新的信号添加内容，将其绑定到按钮上的`enable`属性：

```
RAC(self.signInButton, enabled) = signUpActiveSignal;
```

同样的，我们删除干扰代码，从文件顶部删除两个属性：

```
@property（nonatomic）BOOL passwordIsValid;
@property（nonatomic）BOOL usernameIsValid;
```

删除文本框的action事件：

```
//处理两个文本字段的文本更改 
[ self .usernameTextField addTarget：self 
                           action：@selector（usernameTextFieldChanged）
                 forControlEvents：UIControlEventEditingChanged ];
[ self .passwordTextField addTarget：self  
                           action：@selector（passwordTextFieldChanged）
                 forControlEvents：UIControlEventEditingChanged ];
```

并且删除`updateUIState`、`usernameTextFieldChanged `和`passwordTextFieldChanged `方法，这些都是干扰代码。

运行项目，在用户名和密码文本有效时，登录按钮被启用，就像之前一样。

使用图来可视化新增后的内容：

![内容可视化](https://upload-images.jianshu.io/upload_images/1167313-8939ec3e7e5161dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面说明了一些重要的概念，ReactiveCocoa 可帮您执行一些非常复杂强大的任务；

- **拆分**：信号可以拥有多个订阅者，并作为更多个后续管道的组建来源。在上图中，密码和用户名有效性的布尔信号被拆分并用于文本框背景色和按钮可用两个目的
- **组合**：多个信号可以组合成全新的信号。就如上述例子演示的那样。

ReactiveCocoa 所做的这些更改，使得应用程序不再需要标识文本框当前是否有效的私有属性。这是使用响应式编程和正常编程的关键差异之一--您不再需要使用实例变量来跟踪状态。

**3.2、登录**

应用程序当前使用上面所说的管道来管理文本字段和按钮状态，但是，按钮按下的处理仍然是 actions 事件，因此，我们接下来替换应用程序剩余的逻辑，让整个应用处于被动。

首先我们删除掉 Main.storyboard 中事件的连线。

您已经了解到 ReactiveCocoa 框架如何向标准的 UIKit 控件添加属性和方法。到目前为止，您已经使用过`rac_textSignal`,它会在文本发生变化时发出事件。为了处理按钮事件，您需要使用 ReactiveCocoa 添加到 UIKit 的另一个方法：`rac_signalForControlEvents `。

在 `RWViewController.m`中，添加一下代码：

```
[[self.signInButton rac_signalForControlEvents:UIControlEventTouchUpInside]
 subscribeNext:^(id x) {
     NSLog(@"按钮点击");
 }];
```
上面代码从按钮的 `UIControlEventTouchUpInside ` 事件中创建一个信号，并添加一个订阅，以便在每次发生此事件时输出一个条目信息。

运行项目，在用户名和密码有效时，点击按钮，控制台会会输出结果，这表示我们正确的接收到了按钮的事件。

现在，我们需要将按钮的点击事件和登录验证过程链接起来。`RWDummySignInService.h`中有用来向服务器请求验证的方法：

```
- (void)signInWithUsername:(NSString *)username
                  password:(NSString *)password
                  complete:(RWSignInResponse)completeBlock;
```

该接口将用户名和密码和完成块作为参数，登录验证成功或者失败时都会运行给定的块。您可以直接在按钮的`subscribeNext:`中使用此接口，看似没有上面问题，但是如果这么做的话，依旧属于传统的基于事件行为的。这显然不符合我们新的编程方式。

**3.3、创建异步信号**

将现有的异步API调整为信号非常容易，我们首先删除掉干扰代码`signInButtonTouched:`，在`RWViewController.m`中添加一下方法：

```
-(RACSignal*)signInsignal{
    return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        [self.signInService signInWithUsername:self.usernameTextField.text
                                      password:self.passwordTextField.text
                                      complete:^(BOOL success) {
                                          [subscriber sendNext:@(success)];
                                          [subscriber sendCompleted];
                                      }];
        return nil;
    }];
}
```

上述代码使用了`createSignal:`方法创建信号，该信号的参数是订阅者，当该信号具有订阅者时，块中的代码将被执行。

该块传递一个`subscriber `采用`RACSubscriber `协议的实例，该协议具有发出事件的方法，您也可以发送任意数量的`next`事件，并以`error`和`complete`事件结束。

该块的返回类型是一个`RACDisposable `对象，它允许您执行取消或删除订阅时可能需要的任何清理工作，当信号没有任何清理需求时，返回`nil`即可。

这样，我们现在要利用这个新信号，现在将其加到管道中去。

```
[[[self.signInButton rac_signalForControlEvents:UIControlEventTouchUpInside]
 map:^id(id value) {
     return [self signInsignal];
 }]
 subscribeNext:^(id x) {
     NSLog(@"登录结果：%@",x);
 }];
```

上述代码使用`map`将按钮触摸信号转换为登录信号，订阅者只关注登录结果。

当你运行应用程序，并点击登录按钮，你在控制台中看的的结果将是……
呃呃，好像并不是你期望的那样！

![登录结果](https://upload-images.jianshu.io/upload_images/1167313-04a01346d7883dab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在`subscribeNext:`块接收到的是一个信号对象，而不是登录信号的结果！

![发生了什么？](https://upload-images.jianshu.io/upload_images/1167313-7214a4aa0ae1acb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在`rac_signalForControlEvents`发出`next`事件（与源`UIButton`作为其事件数据）时，点击按钮，`map`创建并返回登录信号，这意味者接下来的信号组建会收到一个`RACSignal`对象，也就是你在`subscribeNext:`中接收到的数据。

上述情况有时被成为信号中的信号，换句话说，包含内部信号的外部信号。如果你目的真的想要这样的信号，你可以在外部信号的`subscribeNext:`块内订阅内部信号，但是这样会导致嵌套变得混乱！幸运的是，这在 ReactiveCocoa 中是一个常见的问题，ReactiveCocoa 已经为此做好了解决方案。

**3.4 Signal of Signals**

为了解决 **Signal of Signals** ，我们需要将`map`更换为`flattenMap`：

```
[[[self.signInButton rac_signalForControlEvents:UIControlEventTouchUpInside]
 flattenMap:^id(id value) {
     return [self signInsignal];
 }]
 subscribeNext:^(id x) {
     NSLog(@"登录结果：%@",x);
 }];
```

这将登录信号映射到按钮的触摸事件，同时将事件从内部信号发送到外部信号来使其`flattens`（平铺化）。

运行应用城西，你将正常使用登录功能：

![登录结果](https://upload-images.jianshu.io/upload_images/1167313-381fd1e3e3345ecb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

完美！

接下来，你将要完成最后一步，在`subscribeNext:`内完成登录成功后的导航：

```
[[[self.signInButton rac_signalForControlEvents:UIControlEventTouchUpInside]
 flattenMap:^id(id value) {
     return [self signInsignal];
 }]
 subscribeNext:^(NSNumber* signedIn) {
     BOOL success = [signedIn boolValue];
     self.signInFailureText.hidden = success;
     if (success) {
         [self performSegueWithIdentifier:@"signInSuccess" sender:self];
     }
 }];
```

运行项目，填写正确的账号密码：user和password，您将看到可爱的小猫！

![成功登录](https://upload-images.jianshu.io/upload_images/1167313-c5f999ba1703722e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**3.5 添加额外操作**

您是否主要当前应用程序存在一个小的用户体验问题呢？在登录验证的，我们应该禁用掉按钮的可用性，防止用户进行重复的登录操作，此外，如果发生登录失败，尝试，则当用户再次尝试登录时，应该隐藏错误消息提示。

但是，您应该如果将此逻辑添加到当前的管道呢？更改按钮的启用状态不是转换，过滤又或者是您到目前为止接触到的其他概念。实际上，它就是所谓的额外操作 “side-effects”，字面意思为副作用。它可以在下个事件发生时在管道中执行一步份逻辑，并且不会改变事件本身的性质。

```
[[[[self.signInButton rac_signalForControlEvents:UIControlEventTouchUpInside]
  doNext:^(id x) {
      self.signInFailureText.hidden = YES;
      self.signInButton.enabled = NO;
  }]
 flattenMap:^id(id value) {
     return [self signInsignal];
 }]
 subscribeNext:^(NSNumber* signedIn) {
     self.signInButton.enabled = YES;
     BOOL success = [signedIn boolValue];
     self.signInFailureText.hidden = success;
     if (success) {
         [self performSegueWithIdentifier:@"signInSuccess" sender:self];
     }
 }];
```

可以看到，`doNet:`在按钮触摸事件创建后立即向管道添加操作步骤。

> `doNext:`块不返回值，因为它是额外的操作，它不会改变事件的性质


新管道可视化：

![添加额外操作](https://upload-images.jianshu.io/upload_images/1167313-94df5a0371cecba3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样，你就完成了全部的业务替换，你的应用程序选在已经完全被动响应式了。

> 在一些异步事件进行时，禁用按钮是一个常见得问题，ReactiveCocoa 再次面临这样的问题。ReactiveCocoa 框架中的 `RACCommand ` 封装了此概念，其中有一个 `enabled`，让您将按钮的启用属性链接到信号。下一部分将介绍此功能

**4.0 小结**

本教程结合小项目循序渐进地向您介绍了ReactiveCocoa的基础用法。在习惯这些概念可能需要一些练习，但就像任何语言或程序一样，一旦掌握了它，它就会变得非常得简单。ReactiveCocoa 得核心概念是`RACSignal`信号，它们只不过是事件流，难道还有什么比这更简单吗？

使用 ReactiveCocoa，我发现得一个有趣得事情是有很多方法可以解决同样得问题。您可能希望尝试使用此demo，通过调整信号和管道以通过将它们拆分和组合得方式完成你想要做得事情。

ReactiveCocoa 得主要目标是使您得代码更清晰，更容易理解。就我个人而言，如果使用函数式响应式语法将业务逻辑表示为清晰得管道数据流，我会发现更容易理解应用程序得功能。





----


## 第二部分

[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) 是一个框架，允许您在 iOS 应用中使用函数式响应式编程（FRP）技术。在上一部分中，您学习两如何使用发出事件流的信号替换标准操作和事件处理逻辑。您还学习两如何转换、分割和组合这些信号。

在本系列的第二部分中，您将了解 ReactiveCocoa 的更高级功能。包括：

- 另外两种事件类型：`error` 和 `completed`
- Throttling
- Threading
- Continuations
- 等等

还等什么，让我们开始吧！

**1.0 Twitter Instant**

您将在本此教程中开发名为**Twitter Instant**的应用程序（以*[Google Instant](http://www.google.co.uk/instant/)*概念为模型），这是一个 Twitter 搜索的应用程序，您可以通过输入文本信息实时更新搜索结果。

此应用程序的[入门项目](https://koenig-media.raywenderlich.com/uploads/2014/01/TwitterInstant-Starter2.zip)包括了基本用户界面和一些基础代码。和第一部分的教程一样，需要您使用[CocoaPods](http://cocoapods.org/)来获取 ReactiveCocoa 框架将其进行集成，集成过程参考上一部分。

运行项目，您将看到一下界面：

![项目界面](https://upload-images.jianshu.io/upload_images/1167313-66681ef9d99755a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是一个非常简单的基于拆分视图控制器`UISplitViewController`的应用程序。左侧面板是** RWSearchFormViewController**，它通过故事版添加了一些UI控件，以及链接到了搜索文本字段，右侧面板是** RWSearchResultsViewController**，它当前只是一个`UITableViewController`的子类，用来展示结果数据。

**2.0 验证搜索文本**

您要做的第一件事就是验证搜索文本以确保其长度大于两个字符。如果你完成了本系列的第一部分，这是一件非常容易做的事情。

在 ** RWSearchFormViewController.m** 中的 `viewDidLoad` 方法添加一下代码：

```
-(BOOL)isValidSearchText:(NSString*)text{
    return text.length > 2;
}
```

该方法目前只是确保搜索字符超过两个字符，这里您或许会问：“这么简单的逻辑，为什么还需要单独方法？”

的确，当前的逻辑很简单，但是如果将来需要更加复杂的处理呢？使用上面的示例，您只需要 在一个地方进行更改即可，此外，上面的代码是您的代码更具表现力，它表明您检查字符长度的原因。我们都应该遵循良好的编码习惯，不是吗？

在文件的顶部导入 ReactiveCocoa:

```
#import <ReactiveCocoa/ReactiveCocoa.h>
```

并添加以下内容：

```
[[self.searchText.rac_textSignal
  map:^id(NSString* text) {
      return [self isValidSearchText:text] ? UIColor.whiteColor : UIColor.yellowColor;
  }]
 subscribeNext:^(UIColor* color) {
     self.searchText.backgroundColor = color;
 }];
```

由上一部分的教程可知道，上述代码使用搜索文本字段的文本信号，将其`map`转换为对应的背景色，然后将其应用于搜索框的背景色。运行测试效果如下，当搜索框输入的文本太短，会显示黄色背景的无效提示。

![输入测试的结果](https://upload-images.jianshu.io/upload_images/1167313-1af5e36604c2f146.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将业务流程进行图形可视化，这个简单的响应式管道看起来就如下图：

![业务可视化](https://upload-images.jianshu.io/upload_images/1167313-c1b45762c3a88233.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在搜索框发生更改时，`rac_textSignal` 会发生**next**事件并包含当前文本值，`map`将文本值转换为颜色信号继续发生**next**事件，而`subscribeNext:`采用此值并将应用于文本框的背景色。

**2.1 管道格式化**

这里需要插一下（反正又不会怀孕），提一个有趣的格式化优化，在进行管道组装时，避免不了一直链接下去，这让代码变得非常的长，阅读起来非常的优雅。ReactiveCocoa 通常的约定是将每一步操作放在新的一行，并垂直对其多有的步骤，就相SwiftUI的那样。

如下图所示，您可以看到一个复杂的示例的对齐方式，来自上一个教程：

![代码格式化](https://upload-images.jianshu.io/upload_images/1167313-65ae163ca9538719.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这使您可以非常轻松地查看构成管道的每一个操作，另外，最小化每一个块中的代码量：任何超过几行的内容都应该分解为私有方法。这个目的是为了让管道结构看起来更清晰，不臃肿，因为数据流即代表您的业务流程。

**2.2 内存管理**

查看您当前在** TwitterInstant**应用程序的代码，您是否想知道您刚刚创建的管道是如何保留的？因为它没有分配给任何变量或属性，它的引用计数不会增加。

ReactiveCocoa 的设计目标之一就是允许管道可以匿名形成这样的编程风格，这让你所有的响应式代码看起来非常的直观。

为了支持这样的模型， ReactiveCocoa 需要维护并保留自己的全局信号集。如果它有一个或者多个订阅者，那么信号有效，如果删除了所有的订阅者，则可以取消分配信号。有关ReactiveCocoa如何管理此过程的更多信息，请参考[内存管理](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/Documentation/MemoryManagement.md)文档。

上面提到一个问题：你如何取消订阅号？在发出一个**completed**或**error**事件后，订阅者将自动删除本身，您也可以通过`RACDisposable`手动移除。

`RACSignal`所有返回的实例`RACDisposable`允许您通过`dispose`方法手动删除订阅者：

```
RACSignal* backgroundColorSignal =
[self.searchText.rac_textSignal
  map:^id(NSString* text) {
      return [self isValidSearchText:text] ? UIColor.whiteColor : UIColor.yellowColor;
  }];

RACDisposable* subscription =
 [backgroundColorSignal subscribeNext:^(UIColor* color) {
     self.searchText.backgroundColor = color;
 }];

// 在未来的某个时刻
[subscription dispose];
```

在实际使用时，您可能不会这样操作，但是手动取消订阅者这点还是值得去了解的。

> 如果你创建一个管道但是没有订阅它，管道永远不会执行，这包括任何的额外操作，如`doNext:`块

**2.3 避免循环引用**

虽然 ReactiveCocoa 在幕后做了很多事情，这并不意味着你无须过多担心信号的内存管理，你只需要考虑一个与内存相关的重要问题。

回顾一下您刚添加的代码：

```
[[self.searchText.rac_textSignal
  map:^id(NSString* text) {
      return [self isValidSearchText:text] ? UIColor.whiteColor : UIColor.yellowColor;
  }]
 subscribeNext:^(UIColor* color) {
     self.searchText.backgroundColor = color;
 }];
```

在`subscribeNext:`块中，`self`获取了文本框的引用，块捕获并保留了范围内的值，因此如果`self`和此信号之间存在强引用，就会导致循环引用的问题。循环引用是否重要取决于`self`对象的生命周期，如果它的生命周期和应用程序的持续时间一致，就像现在的情况，那它并不重要，但是在开发复杂的应用程序过程中，大部分的`self`都是需要避免循环引用的。

为了解决上述问题，使用[Blocks](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html#//apple_ref/doc/uid/TP40011210-CH8-SW16)的Apple文档建议捕获弱引用`self`，就像下面这样：

```
__weak RWSearchFormViewController* bself = self;// 捕获弱引用
[[self.searchText.rac_textSignal
  map:^id(NSString* text) {
      return [self isValidSearchText:text] ? UIColor.whiteColor : UIColor.yellowColor;
  }]
 subscribeNext:^(UIColor* color) {
     bself.searchText.backgroundColor = color;
 }];
```

在上面的代码`bself`引用了`self`，但是其已经被标记为`__weak`使其弱引用。但是这看起来不太优雅。

ReactiveCocoa 框架提供了一个宏，用来代替上面的代码，您可以导入下面代码来使用：

```
#import <ReactiveCocoa/RACEXTScope.h>
```

然后使用下面代码替换上面的代码：

```
@weakify(self)
[[self.searchText.rac_textSignal
  map:^id(NSString* text) {
      return [self isValidSearchText:text] ? UIColor.whiteColor : UIColor.yellowColor;
  }]
 subscribeNext:^(UIColor* color) {
     @strongify(self)
     self.searchText.backgroundColor = color;
 }];
```

上面的`@weakify`和`@strongify`语句是在[Extended Objective-C](https://github.com/jspahrsummers/libextobjc)库中定义的宏，它们也包含在ReactiveCocoa中。

> 如果你有兴趣想知道`@weakify`和`@strongify`实际上做了什么，Xcode中选择Product -> Perform Action -> Preprocess “RWSearchForViewController”。这将预处理视图控制器，展开多有宏并允许您查看最终输出。

最后要注意的是，在块中使用实例变量（如_name）时要小心，这些也同样会导致块捕获强引用`self`。如果代码上导致此问题，您可打开编译器警告来提醒您。在项目的构建设置中搜索**retain**查找：

![retain](https://upload-images.jianshu.io/upload_images/1167313-70a575739d4c9f82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 在上一个教程中，敏锐的读者肯定主要到`subscribeNext:`可以使用`RAC`宏来消除当前管道中块的引用。如果您发现了这一点，那么就作出改变并给自己一个闪亮的星星✨！

好了，一些优化和注意事项以完成，下面接着为项目添加一些真正的功能！

**3.0 访问Twitter**

您将使用社交框架来允许应用程序可以搜索推文。在添加代码之前，您需要在运行此应用程序的模拟器或者iPad中登录您的Twitter账号：

![登录Twitter账号](https://upload-images.jianshu.io/upload_images/1167313-858bd7631337cd07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用cocoaPods导入所需要的社交框架，接着在** RWSearchFormViewController**中，导入文件头：

```
#import <Accounts / Accounts.h>
#import <Social / Social.h>
```

在导入的下方添加以下几个枚举和常量：

```
typedef NS_ENUM(NSInteger, RWTwitterInstantError){
    RWTwitterInstantErrorAccessDenied,
    RWTwitterInstantErrorNoTwitterAccounts,
    RWTwitterInstantErrorInvalidResponse
};

static  NSString * const RWTwitterInstantDomain = @"TwitterInstant";
```

在同一文件的下方，声明以下属性：

```
@property (strong, nonatomic) ACAccountStore * accountStore;
@property (strong, nonatomic) ACAccountType * twitterAccountType;
```

在`viewDidLoad `中添加：

```
self.accountStore = [[ACAccountStore alloc] init];
self.twitterAccountType = [self.accountStore
  accountTypeWithAccountTypeIdentifier：ACAccountTypeIdentifierTwitter];
```

这将创建账户存储和Twitter账号标识符。

应用请求访问社交媒体账户是一个异步操作，因此我们需要使用一个异步信号将其包装起来，以便之后使用它！

```
-(RACSignal*)requestAccessToTwitterSignal{
    // 定义错误
    NSError* accessError = [NSError errorWithDomain:RWTwitterInstantDomain code:RWTwitterInstantErrorAccessDenied userInfo:nil];
    // 创建信号
    @weakify(self)
    return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        // 访问twitter
        @strongify(self)
        [self.accountStore requestAccessToAccountsWithType:self.twitterAccountType options:nil completion:^(BOOL granted, NSError *error) {
             // 处理response
             if (!granted) {
                 [subscriber sendError:accessError];//访问被拒绝，发出错误事件
             } else {
                 [subscriber sendNext:nil];
                 [subscriber sendCompleted];
             }
         }];
        return nil;
    }];
}
```

上述方法执行了以下操作：

1. 定义了一个错误，如果用户被拒绝访问，则会发送该错误；
2. 使用`createSignal`创建了一个实例`RACSignal`;
3. 通过请求访问Twitter
4. 在用户被授予或拒绝访问之后，发送对应的信号事件。如果用户被授予访问，则发送**next**事件，然后发送完成。如果用户被拒绝访问，则会发送**error**事件。

在信号的生命周期内，它可以不发出任何事件，一个或多个事件，然后是**completed**事件和**error**事件。

现在，我们来使用此信号：

```
[[self requestAccessToTwitterSignal]
 subscribeNext:^(id x) {
     NSLog(@"授予访问权限");
 } error:^(NSError *error) {
     NSLog(@"发生错误：%@",error);
 }];
```

运行应用程序，您将看到以下内容：

![运行结果](https://upload-images.jianshu.io/upload_images/1167313-0f417d9b97f74734.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果点击“ok”，则`subscribeNext:`块中的日志消息会出现在控制台中，相反，则会执行错误块中的消息。

社交框架会记住你选择的决定，然后需要您重新输入您的Twitter账号密码。

**4.0 链接信号**

一旦用户被授予访问其Twitter账户的权限，应用程序需要持续监视搜索文本框的内容，以便查询twitter。

应用程序需要等待访问Twitter的信号发出其已完成的事件，然后订阅文本框字段的信号，这里涉及到不同信号的顺序链接的问题，ReactiveCocoa 非常优雅地处理这个问题。

```
[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
      @strongify(self)
      return self.searchText.rac_textSignal;
  }]
 subscribeNext:^(id x) {
     NSLog(@"%@",x);
 } error:^(NSError *error) {
     NSLog(@"发生错误：%@",error);
 }];
```

`then` 方法可以等待`RACSignal`发出**completed**事件，然后订阅其block参数返回的信号，这有效地控制从一个信号传递下一个信号。

> 您之前对`self`已经进行来弱引用，因此这里只添加来一个`@strongify(self)`。

该`then`方法会传递**error**事件，因此最终`subscribeNext:error:`块仍然接收初始化访问请求发出的错误。

接下来，使用`filter`向管道添加过滤条件以删除任何无效的搜索字符串：

```
[[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
      @strongify(self)
      return self.searchText.rac_textSignal;
  }]
  filter:^BOOL(NSString* text) {
      @strongify(self)
      return [self isValidSearchText:text];
  }]
 subscribeNext:^(id x) {
     NSLog(@"%@",x);
 }
 error:^(NSError *error) {
     NSLog(@"发生错误：%@",error);
 }];
```

使用图形来可视化当前应用程序的管道组成：

![当前管道](https://upload-images.jianshu.io/upload_images/1167313-5eb1ab1eb524d2f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

应用程序管道从`requestAccessToTwitterSignal`开始，然后切换到`rac_textSignal`，通过一个过滤器，最终到达订阅块。不仅如此，您还可以看到第一步发生的任何错误事件都会被同一个`subscribeNext:error:`块占用。

**5.0 搜索Twitter**

Social Framework 用来访问搜索Twitter，然而，该框架不应是响应式的，下一步要做的就是将所需的API方法调用包装在一个信号中。

首先使用文本字段包装为一个请求对象：

```
- (SLRequest *)requestforTwitterSearchWithText:(NSString *)text {
    NSURL *url = [NSURL URLWithString:@"https://api.twitter.com/1.1/search/tweets.json"];
    NSDictionary *params = @{@"q" : text};
    SLRequest *request =  [SLRequest requestForServiceType:SLServiceTypeTwitter
                                             requestMethod:SLRequestMethodGET
                                                       URL:url
                                                parameters:params];
    return request;
}
```

接着，我们将使用此请求生成信号：

```
- (RACSignal *)signalForSearchWithText:(NSString *)text {
    // 定义错误
    NSError *noAccountsError = [NSError errorWithDomain:RWTwitterInstantDomain
                                                   code:RWTwitterInstantErrorNoTwitterAccounts
                                               userInfo:nil];
    NSError *invalidResponseError = [NSError errorWithDomain:RWTwitterInstantDomain
                                                        code:RWTwitterInstantErrorInvalidResponse
                                                    userInfo:nil];
    
    // 创建信号
    @weakify(self)
    return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        @strongify(self);
        // 创建请求
        SLRequest *request = [self requestforTwitterSearchWithText:text];
        // 提供推特账号
        NSArray *twitterAccounts = [self.accountStore
                                    accountsWithAccountType:self.twitterAccountType];
        if (twitterAccounts.count == 0) {
            [subscriber sendError:noAccountsError];
        } else {
            [request setAccount:[twitterAccounts lastObject]];
            // 开始请求
            [request performRequestWithHandler: ^(NSData *responseData,
                                                  NSHTTPURLResponse *urlResponse, NSError *error) {
                if (urlResponse.statusCode == 200) {
                    // 成功，解析响应
                    NSDictionary *timelineData =
                    [NSJSONSerialization JSONObjectWithData:responseData
                                                    options:NSJSONReadingAllowFragments
                                                      error:nil];
                    [subscriber sendNext:timelineData];
                    [subscriber sendCompleted];
                }
                else {
                    // 错误，发送错误
                    [subscriber sendError:invalidResponseError];
                }
            }];
        }
        return nil;
    }];
}
```

有关请求的信号已经创建完成，现在来使用这个新信号吧！

在本教程的第一部分，您学习了如何使用`flattenMap`将每个**next**事件映射到随后订阅的新信号，现在我们再次需要使用它了。

```
[[[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
      @strongify(self)
      return self.searchText.rac_textSignal;
  }]
  filter:^BOOL(NSString* text) {
      @strongify(self)
      return [self isValidSearchText:text];
  }]
 flattenMap:^RACStream *(NSString* text) {
     return [self signalForSearchWithText:text];
 }]
 subscribeNext:^(id x) {
     NSLog(@"%@",x);
 }
 error:^(NSError *error) {
     NSLog(@"发生错误：%@",error);
 }];
```

运行应用程序，然后在搜索框中输入一些文本。一旦文本长度超过两个字符时，控制台可以看到Twitter搜索的结果。

以下仅显示您将看到的数据类型的片段：

```
TwitterInstant[40308:5403] {
    "search_metadata" =     {
        "completed_in" = "0.019";
        count = 15;
        "max_id" = 419735546840117248;
        "max_id_str" = 419735546840117248;
        "next_results" = "?max_id=419734921599787007&q=asd&include_entities=1";
        query = asd;
        "refresh_url" = "?since_id=419735546840117248&q=asd&include_entities=1";
        "since_id" = 0;
        "since_id_str" = 0;
    };
    statuses =     (
                {
            contributors = "<null>";
            coordinates = "<null>";
            "created_at" = "Sun Jan 05 07:42:07 +0000 2014";
            entities =             {
                hashtags = ...
```

>  - 同样是信号中的信号，`then:`的作用是链接到新的信号，新信号和之前的信号可以没有订阅，`flattenMap:`则将每个**next**事件映射到随后订阅的中，例如接收了用户输入的文本信息。
> - 关于**error**事件：一旦信号发出错误，它就会直接进入错误处理块，而不会继续执行管道中接下来的订阅块
> - 当Twitter请求返回错误时，请继续执行其他异常流程。

**6.0 线程**

在您急不可耐的想要将Twitter搜索到的JSON显示到界面上之前，有些事情你需要知道：ReactiveCocoa 将管道中的每一个操作元素都执行在不同的线程上，而我们更新UI都需要切换到主线程。那么我们该如何更新UI呢？典型的方法是使用操作队列等，但是ReactiveCocoa有一个更简单的方法来解决切换线程的问题。

```
[[[[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
      @strongify(self)
      return self.searchText.rac_textSignal;
  }]
  filter:^BOOL(NSString* text) {
      @strongify(self)
      return [self isValidSearchText:text];
  }]
 flattenMap:^RACStream *(NSString* text) {
     return [self signalForSearchWithText:text];
 }]
 deliverOn:RACScheduler.mainThreadScheduler]
 subscribeNext:^(id x) {
     NSLog(@"%@",x);
 }
 error:^(NSError *error) {
     NSLog(@"发生错误：%@",error);
 }];
```

这样，您就可以轻松的在主线中更新UI了。

> 如果您查看`RACScheduler`类，您将看到有很多选项可用于具有不同优先级的线程，又或者在管道中添加延迟。

**6.1 更新UI**

如果你打开`RWSearchResultsViewController.h`你会看到它已经有一个`displayTweets:`方法，这可以让右侧视图控制器呈现提供的推文数组。实现非常简单，它只是一个标准`UITableView`数据源，该`displayTweets:`方法的单个参数需要`NSArray`包含`RWTweet`实例。

事件到达`subscibeNext:error:`的数据是一个字典，如何解析这个字典呢？该字典中包含了一个key为`statuses`的数组，这是微博数组，也是一个字典实例。

正好`RWTweet`已经有了一个类方法`tweetWithStatus:`来获取字典中数据，那么，您所需要做的就是编写一个for循环，然后迭代数组，为每个推文创建一个`RWTweet`实例。

但是，你无需这样做，这里推荐一个更优雅的数据转换库[LinqToObjectiveC](https://github.com/ColinEberhardt/LinqToObjectiveC) 。现在更新Podfile文件:

```
pod'LinqToObjectiveC'，'2.0.0'
```

执行更新操作，将框架拉取下来。

打开`RWSearchFormViewController.m`并将以下导入添加到文件的顶部：

```
#import “RWTweet.h”
#import “NSArray + LinqExtensions.h”
```

`NSArray+LinqExtensions.h`允许您变换、排序、分组和过滤其数据。

使用此API，更新当前管道：

```
[[[[[[self requestAccessToTwitterSignal]
  then:^RACSignal *{
      @strongify(self)
      return self.searchText.rac_textSignal;
  }]
  filter:^BOOL(NSString* text) {
      @strongify(self)
      return [self isValidSearchText:text];
  }]
 flattenMap:^RACStream *(NSString* text) {
     return [self signalForSearchWithText:text];
 }]
 deliverOn:RACScheduler.mainThreadScheduler]
 subscribeNext:^(NSDictionary * jsonSearchResult) {
     NSArray* statuses = jsonSearchResult[@"status"];
     NSArray* tweets = [statuses linq_select:^id（id tweet）{
         return [RWTweet tweetWithStatus:tweet];
     }];
     [self.resultsViewController displayTweets:tweets];
 }
 error:^(NSError *error) {
     NSLog(@"发生错误：%@",error);
 }];
```

该`subscribeNext:`块首先获取到推文的数组，方法`linq_select`将每个数组元素转换生成`RWTweet`实例。最后将转换后的推文发送到结果视图控制器。

最终看到UI中出现的推文：

![最终结果](https://upload-images.jianshu.io/upload_images/1167313-7f0f048e5c83f645.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**6.2 图像异步加载**

可能你已经注意到每条推文的左侧都又一个间隙，实际上左边是用户的头像。

`RWTweet`中已经有了一个`profileImageUrl`属性，用户存储图像合适的url地址。为了能够使表格平滑的滚动，您需要确保在主线程上不执行耗时操作，如此处图像的获取。通常，您可以使用GCD和NSOprationQueue来实现，但是，为什么不"问问"神奇的ReactiveCocoa呢？

```
-(RACSignal*)signalForLoadingImage:(NSString*)imageUrl{
    RACScheduler* scheduler = [RACScheduler schedulerWithPriority:RACSchedulerPriorityBackground];
    return [[RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        NSData* data = [NSData dataWithContentsOfURL:[NSURL URLWithString:imageUrl]];
        UIImage* image = [UIImage imageWithData:data];
        [subscriber sendNext:image];
        return nil;
    }] subscribeOn:scheduler];
}
```

上述方法首先获得后台调度程序，因为您希望此信号在主线程以外的线程上执行。接着，它会创建一个信号，用于下载图像数据并将UIImage传递出去，最后使用`subscribeOn:`确保信号在给定的调度程序上执行。

现在在代理方法`tableView:CellForRowAtIndex:`中添加图像的获取：

```
cell.twitterAvatarView.image = nil ;

[[[ self signalForLoadingImage:tweet.profileImageUrl]
  deliverOn:[RACScheduler mainThreadScheduler]]
  subscribeNext:^(UIImage * image){
   cell.twitterAvatarView.image = image;
  }];

```

`deliveOn:`确保您**next**事件在指定线程上执行。

![正确显示头像](https://upload-images.jianshu.io/upload_images/1167313-0df4b635060dd076.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**7.0 Throttling 节流**

您可能已经注意到，在每次输入或者删除字符时都会执行搜索功能，这可能导致应用程序在每秒都会执行多次搜索，这显然是不理想的：用户在输入字符未结束时，搜索的结果通常并不是用户想要，而且会被下一次的搜索给丢弃。其次，这样还会消耗用户大量的流量以及给服务器造成压力。

更好的一种做法就是限制信号的频率，这里使用时间来限制，例如500毫秒内保持不变时再执行搜索。

```
[[[[[[[self requestAccessToTwitterSignal]
      then:^RACSignal *{
          @strongify(self)
          return self.searchText.rac_textSignal;
      }]
     filter:^BOOL(NSString* text) {
         @strongify(self)
         return [self isValidSearchText:text];
     }]
    throttle:0.5]
   flattenMap:^RACStream *(NSString* text) {
     return [self signalForSearchWithText:text];
   }]
  deliverOn:RACScheduler.mainThreadScheduler]
 subscribeNext:^(NSDictionary * jsonSearchResult) {
     NSArray* statuses = jsonSearchResult[@"status"];
     NSArray* tweets = [statuses linq_select:^id（id tweet）{
         return [RWTweet tweetWithStatus:tweet];
     }];
     [self.resultsViewController displayTweets:tweets];
 }
 error:^(NSError *error) {
     NSLog(@"发生错误：%@",error);
 }];
```

如果在给定的时间段未收到另外一个**next**事件，则`throttle`操作才会发送**next**事件。

运行应用程序，您在输入500毫秒之后，搜索结果才会更新，这让用户感觉好了很多不是吗？

[最终的项目](https://koenig-media.raywenderlich.com/uploads/2014/01/TwitterInstant-Final.zip)

**8.0 总结**

来欣赏以下根据业务需求拼装的管道最终样子：

```
@weakify(self)
// 1.权限请求信号
[[[[[[[self requestAccessToTwitterSignal]
      // 2.链接新的文本信号
      then:^RACSignal *{
          @strongify(self)
          return self.searchText.rac_textSignal;
      }]
     // 3.过滤验证文本数据
     filter:^BOOL(NSString* text) {
         @strongify(self)
         return [self isValidSearchText:text];
     }]
    // 时间限制流
    throttle:0.5]
   // 4.Twiteer搜索信号
   flattenMap:^RACStream *(NSString* text) {
     return [self signalForSearchWithText:text];
   }]
  // 切换线程
  deliverOn:RACScheduler.mainThreadScheduler]
 // 5.处理搜索结果
 subscribeNext:^(NSDictionary * jsonSearchResult) {
     NSArray* statuses = jsonSearchResult[@"status"];
     NSArray* tweets = [statuses linq_select:^id(id tweet){
         return [RWTweet tweetWithStatus:tweet];
     }];
     [self.resultsViewController displayTweets:tweets];
 }
 // 发生错误时
 error:^(NSError *error) {
     NSLog(@"发生错误：%@",error);
 }];
```

在这里你可以看到完成的业务流程，即数据流向。

![管道图例](https://upload-images.jianshu.io/upload_images/1167313-474ac8d344b1f03a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是一个相当复杂的数据流，所有这些都简洁地表达为单个响应流水线。你能想象这个业务流程使用非响应技术去实现会有多复杂吗？实现完成之后想要看到数据流会有多困难，不是吗？现在你不必再走原来的那条路了！现在你知道ReactiveCocoa有多棒了！

> 值得一提的是，ReactiveCocoa 可以应用在MVVM设计模式上，它可以更好地分离应用程序逻辑和视图逻辑。
















