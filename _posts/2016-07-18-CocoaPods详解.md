---
layout:     post
title:      CocoaPods详解：安装和使用以及可能遇到的错误
subtitle:   新手快速入门
date:       2016-07-18
author:     LOLITA0164
header-img: img/post-bg-project.jpg
catalog: true
tags:
    - iOS
    - 项目管理
---

## CocoaPods是什么

 1、问题
 
 在IOS开发过程中，经常会使用到第三方框架，通常都是去GitHub上下载相关的框架，导入工程并添加框架所依赖的framework，那么问题就出现了，如何解决工程所依赖的framework的重复性，并且当三方框架更新时，需要我们手动的删除旧的框架、重新下载框架的最新版本，添加依赖的库，这过程繁琐而又易错。那有没有什么好的工具来解决上述的问题呢？答案是有的，就是CocoaPods。
 
 2、CocoaPods是什么

 CocoaPods是类库管理工具，IOS开发过程中遇到的两个问题——依赖库的重复性和三方框架的更新，使用cocoaPods管理工具，只需要一行命令就可以完全解决，并且绝大部分有名的开源类库，都支持CocoaPods。因此，在开发IOS应用时，掌握CocoaPods的使用是必不可少的基本技能。
使用CocoaPods所生成的workspace能够让我们能方便直观的管理第三方开源库。

## 安装CocoaPads

安装CocoaPods需要Ruby环境，OS X系统默认的已经可以运行Ruby，因此我们只需要执行以下命令来安装CocoaPods：

```
$ sudo gem install cocoapods
```

安装结束后，执行以下命令来初始化(下载服务器中所有第三方框架信息, 缓存到电脑本地)

```
$ pod setup
```

以上就完成了cocoapods的安装
 
**安装中出现的问题**

-  **执行sudo gem install cocoapods  没反应**
因为Ruby的默认源使用的是https://rubygems.org/， 国内访问这个网址会有问题，这里需要使用换成国内的taobao镜像服务器，并且因为iOS9.0只支持HTTPS，所以以前不能用了将这里改成HTTPS即可，替换方式如下：

```
$gem sources --remove https://rubygems.org/
$gem sources -a https://ruby.taobao.org/
```

验证是否替换成功：

```
gem sources -l
```

- **执行pod setup出错,**
出现 Setting up CocoaPods master repo，我第一次初始化pod时，出现了这个错误,未能执行成功，联想到之前的换源问题，我试着将https://ruby.taobao.org/ 替换成了 http://ruby.taobao.org/， 最终成功初始化。
 
- **Ruby的版本过低**
出现active support requires Ruby version >=2.2.0字样（目前Mac系统自带的事1.8.7版本），此时需要更新Ruby版本，至少2.2以上，这里推荐使用RVM（Ruby Version Manager，Ruby版本管理器）来升级Ruby。
安装RVM
```
$curl -L get.rvm.io | bash -s stable
```
然后执行
```
$source ~/.bashrc
$source ~/.bash_profile
```
升级Ruby
```
//查看当前版本
$ ruby -v  
//列出各个版本信息
$ rvm list known rubies 
//安装对应版本
$rvm install x.x.x
```

- **安装第三方框架**
```
//会根据Podfile.lock文件记录的版本号, 去下载对应版本的第三方框架
$ pod install
```

- **升级第三方框架**
```
//如果Podfile中, 第三方框架没有明确声明版本号, 就会自动将第三方框架升级到最新版本, 并且更新Podfile.lock文件 
$ pod update
```

## 使用CocoaPads
以下演示CocoaPads的使用过程，我们先新建一个工程：cocoapodsDemo。
1. 新建Podfile文件 
目录结构如下：
![这里写图片描述](https://ws2.sinaimg.cn/large/006tNbRwgy1fwi8hacmc2j30ku0bqwfe.jpg)
2. 编辑Podfile文件，以AFNetworking，JSONKit，Reachability为例，编辑后的内容如下：
![这里写图片描述](https://ws3.sinaimg.cn/large/006tNbRwgy1fwi8hqr7j0j30dw05kwfa.jpg)
3. 使用cocoa pods下载和配置三方
在**终端**执行以下命令：
```
//先进入cocoapodsDemo项目文件
$ cd desktop/cocoapodsDemo
//执行安装命令
$ pod install
```
安装的结果如下：
![这里写图片描述](https://ws4.sinaimg.cn/large/006tNbRwgy1fwi8idtefmj30v80aitba.jpg)
 4. 重新打开我们的项目
![安装后的目录](https://ws4.sinaimg.cn/large/006tNbRwgy1fwi8inkcwtj30z60dydhp.jpg)
对比之前，目录中多出了三个文件，此时我们如果运行cocoapodsDemo.xcodeproj,会出现错误，我们打开cocoapods为我们产生的workspace文件：
![这里写图片描述](https://ws1.sinaimg.cn/large/006tNbRwgy1fwi8j0r17hj309q0ammxs.jpg)
cocoapodsDemo是我们原来建的项目工程
需要说明的是：
1.第三方库已经被编译成静态库供的工程使用
![这里写图片描述](https://ws4.sinaimg.cn/large/006tNbRwgy1fwi8jc1me3j31k207a75q.jpg)
2.CocoaPods将所有的第三方库以target的方式组成一个名为Pods的工程里，我们的工程和第三方库所在的工程Pods由一个新生成的workspace管理
![这里写图片描述](https://ws1.sinaimg.cn/large/006tNbRwgy1fwi8jiurmyj30em05smxh.jpg)

**补充说明**

- 添加新的框架
添加框架时只需要在Podfile中新添加字段，并在终端重新执行命令即可
```
$pod install
```
- 获取三方框架信息
在终端执行以下命令即可搜索第三方框架信息,如
```
pod search AFNetworking
```
- 更新框架
```
pod update
```


## 可能遇到的错误

1、确保你所pod的三方库和你的target系统版本匹配；

2、发生一下错误
```
[!] The `test [Debug]` target overrides the `OTHER_LDFLAGS` build setting defined in `Pods/Target Support Files/Pods/Pods.debug.xcconfig'. This can lead to problems with the CocoaPods installation
- Use the `$(inherited)` flag, or
- Remove the build settings from the target.
```

产生此警告的原因是项目 Target 中的一些设置，CocoaPods 也做了默认的设置，如果两个设置结果不一致，就会造成问题。

- 分别在我的项目中定义PODS_ROOT 和 Other Linker Flags的地方，把他们的值用$(inherited)替换掉，进入终端，执行pod install
- 点击项目文件 project.xcodeproj，右键显示包内容，用文本编辑器打开project.pbxproj，删除OTHER_LDFLAGS的地方保存，pod install


----------


**相关链接**

- [关于RVM](http://ruby-china.org/wiki/rvm-guide)
- [安装CocoaPods遇过的坑](http://www.th7.cn/Program/Ruby/201609/966343.shtml)
