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


















