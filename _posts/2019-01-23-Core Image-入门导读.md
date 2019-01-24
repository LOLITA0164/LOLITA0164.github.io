---
layout:     post
title:      Core Image - 入门导读
subtitle:   关于图片滤镜的简单介绍，快速上手
date:       2018-12-26
author:     LOLITA0164
header-img: img/post-bg-coreImage.jpg
catalog: true
tags: 
    - Core Image
    - 滤镜
---

![高斯模糊](https://ws1.sinaimg.cn/large/006tNc79gy1fzgg9mx2avj30p30b9gln.jpg)

## 简介

Core Image 是一种图像处理和分析技术，可为静态和视频图像提供高性能处理。 使用许多内置图像过滤器来处理图像并通过链接过滤器来构建复杂效果。

关于过滤器种类和效果可以查看官方文档：[Core Image Filter Reference](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/uid/TP40004346)

系统内置了多种滤镜效果，你可以将多种滤镜组合使用，一般情况下的需求都可以满足。当然，你也可以自定义滤镜效果。

关于自定义了滤镜可以查看官方文档：[Core Image Programming Guide](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/CoreImaging/ci_intro/ci_intro.html#//apple_ref/doc/uid/TP30001185)

## 基础知识

#### 三个重要的类

- [CIImage](https://developer.apple.com/documentation/coreimage/ciimage) 

Core Image 中的图像类，类似 UIKit 中的 UIImage 类

- [CIFilter](https://developer.apple.com/documentation/coreimage/cifilter)

Core Image 中的图像过滤器，可以使用它来处理一个或者多个图像，并产生新的图像数据

- [CIContext](https://developer.apple.com/documentation/coreimage/cicontext)

Core Image 中处理图像的上下文，可以用来渲染、分析图像

> CIContext 和 CIImage 对象是不可变的，因此多个线程可以使用同一个 CIContext 对象来呈现 CIImage 对象。但是，CIFilter 对象是可变的，因此无法在线程之间安全地共享，每个线程必须创建自己的 CIFilter 对象。

#### 应用在过滤器中的参数类型

- CIColor

定义了特定颜色空间

- CIVector

坐标值，方向矢量，矩阵和其他非标量值的容器，用于 Core Image 中的过滤器参数

- CIFilterShape

过滤器的边界形状和过滤操作的定义域的描述

- CIFormat

图像输入，输出和处理的像素数据格式


## 内置过滤器的使用示例

![三种过滤器](https://ws2.sinaimg.cn/large/006tNc79gy1fzgl9w7y7pj31440dqwg2.jpg)

上图使用了三种过滤器组合起来实现效果：

1、使用 sepia 对带有红褐色色调的图像进行着色

2、添加 bloom 滤镜以突出显示高光

3、使用 Lanczos 比例滤镜缩小图像

接下来我们来演示一下滤镜的使用步骤。

**步骤一：上下文和资源图像**

- 上下文

正像我们之前提到的，我们需要一块处理图像的上下文：CIContext ，创建该对象是非常消耗资源的，因此我们可以在应用的整个生命周期中一直持有该对象重复使用。

```
let context = CIContext()
```

- 图像

巧妇难为无米之炊，我们还需要提供被处理的图像资源：CIImage ，该对象本身是不可显示的，你可以将其转换为 UIImage 进行显示，或者将 UIImage 转为 CIImage 对象。

```
let imageURL = URL(fileURLWithPath: "\(Bundle.main.bundlePath)/YourImageName.png")

// CIImage 对象
let originalCIImage = CIImage(contentsOf: imageURL)!

// 转换为 UIImage 对象显示
self.imageView.image = UIImage(ciImage:originalCIImage)
```

![原始图片](https://ws4.sinaimg.cn/large/006tNc79gy1fzgne02ed2j305k080mxr.jpg)


**步骤二：选择内置过滤器**

内置过滤器是不可见属性的类型，我们可以使用 KVC 的形式提前给过滤器设置过滤器的相关属性，也可以在创建过滤器时指定。如果你不知道有哪些过滤器，可以查询官方文档：[Core Image Filter Reference]()。

- 使用 Sepia 过滤器

![sepia 过滤器](https://ws1.sinaimg.cn/large/006tNc79gy1fzgmkt8tudj30u00v2q95.jpg)

系统提供了一些常见的和参数相关的key，例如 `kCIInputImageKey`、`kCIInputRadiusKey` 等，如果你无法推断关联的 key，则只需使用过滤器对应参数的字符串即可。例如上图中的 `inputImage` 就表示 `kCIInputImageKey`。

回到之前的例子，我们来创建 sepia 过滤器过滤图片。

```
func sepiaFilter(_ input: CIImage, intensity: Double) -> CIImage?
{
    // 创建过滤器
    let sepiaFilter = CIFilter(name:"CISepiaTone")
    // 设置输入图片
    sepiaFilter?.setValue(input, forKey: kCIInputImageKey)
    // 设置强度
    sepiaFilter?.setValue(intensity, forKey: kCIInputIntensityKey)
    // 返回输出值
    return sepiaFilter?.outputImage
}
```

这样我们就可以使用该方法得到 sepia 过滤器过滤后图片。

```
let sepiaCIImage = sepiaFilter(ciImage!, intensity:0.9)
```

我们将 sepiaCIImage 转换为 UIImage 进行显示。

```
self.imageView.image = UIImage.init(ciImage: sepiaCIImage!)
```

![sepia 过滤后图片](https://ws1.sinaimg.cn/large/006tNc79gy1fzgnemfk0wj305k080q3o.jpg)

以上就完整演示了内置滤镜的步骤。当然，我们还可以接着对已过滤的图像进一步使用过滤器。

为了完整演示示例中的效果，我们接着 使用 Bloom 过滤器，Bloom 过滤器突出了图像的高光部分。

- 使用 Bloom 过滤器 

![Bloom 过滤器](https://ws1.sinaimg.cn/large/006tNc79gy1fzgnm4o3pnj30u00xstf6.jpg)

像 Sepia 滤镜一样，Bloom 滤镜效果的强度范围在0到1之间，其中1是最强烈的效果。Bloom 过滤器具有额外的 inputRadius 参数，以确定发光区域将扩展多少。可以使用范围值来微调效果，或者将输入参数分配给控件（如 UISlider ）以允许用户调整其值。

```
func bloomFilter(_ input:CIImage, intensity: Double, radius: Double) -> CIImage?
{
    let bloomFilter = CIFilter(name:"CIBloom")
    bloomFilter?.setValue(input, forKey: kCIInputImageKey)
    bloomFilter?.setValue(intensity, forKey: kCIInputIntensityKey)
    // 设置发光区域
    bloomFilter?.setValue(radius, forKey: kCIInputRadiusKey)
    return bloomFilter?.outputImage
}
```

> 注：CIGloom 过滤器可以执行和 Bloom 过滤器相反的效果

```
let bloomCIImage = bloomFilter(sepiaCIImage!, intensity:1, radius:10)
self.imageView.image = UIImage(ciImage:bloomCIImage!)
```

同样的，我们将图片转换为 UIImage 进行显示。

![Bloom 过滤后图片](https://ws1.sinaimg.cn/large/006tNc79gy1fzgnt727ivj305k085747.jpg)

注：实际上，通过这一步中的 Bloom 滤镜 会将图片填充莫名其妙的白色部分，并且整体图像尺寸要比原图尺寸大一些，就如下图所示。

![莫名其妙的填充](https://ws3.sinaimg.cn/large/006tNc79gy1fzhfvmd5ptj305k09naah.jpg)

细心的读者应该发现，我们之前创建的 CIContext 对象在整个滤镜过程中并未使用到对象 Context，而在 CIFilter 对象 outputImage 得到 CIImage 对象后，就使用 `init(ciImage: CIImage)` 方法直接转换为 UIImage 对象了（实际上这种转换是有系统自动创建 CIContext 对象完成的，但是创建 CIContext 对象是非常消耗资源的，因此并不推荐频繁的创建该对象）。要知道，在内置的许多过滤器的输出图像和输入图像具有不同的尺寸比。例如，一个模糊图像（如高斯模糊）由于采样超出了输入图像的边缘，围绕在其边界外还会有一些额外的像素，另外，Core Image 是一个 OS X 和 iOS 的图像处理框架，它采用了前者的坐标系，因此上图会出现图片整体在左下角的情况。这时候，CIContext 对象就起到了作用，我们需要 CIContext 从输出图像的一个矩形内（bounds）创建一个 CGImage。

修改之前的代码：

```
let bloomCIImage = bloomFilter(sepiaCIImage!, intensity:1, radius:10)
let image2 = Context.createCGImage(bloomCIImage!, from: (ciImage?.extent)!)
self.imageView.image = UIImage.init(cgImage: image2!)
```

这样我们就可以得到预期的图像：

![Context转换后的图像](https://ws2.sinaimg.cn/large/006tNc79gy1fzhh9b9ifwj305k09qt9b.jpg)





- 缩放图片

![CILanczosScaleTransform](https://ws4.sinaimg.cn/large/006tNc79gy1fzgo4gtc20j30u0102jws.jpg)

使用 CILanczosScaleTransform 可以高质量下重新采样获得新图像，通过CILanczosScaleTransform 滤镜的输入参数 aspectRatio 为 1 时，可保留原始图像的纵横比，你也可以传入[0,1]其他值，如0.5，结果将是：宽度缩小为一半，图像纵向拉伸。

```
func scaleFilter(_ input:CIImage, aspectRatio : Double, scale : Double) -> CIImage
{
    let scaleFilter = CIFilter(name:"CILanczosScaleTransform")!
    scaleFilter.setValue(input, forKey: kCIInputImageKey)
    // 设置目标大小
    scaleFilter.setValue(scale, forKey: kCIInputScaleKey)
    // 目标图像的宽高之比
    scaleFilter.setValue(aspectRatio, forKey: kCIInputAspectRatioKey)
    return scaleFilter.outputImage!
}
```

这样我们将图像的宽度缩小一半，图像整体缩小一半。

```
let scaledCIImage = scaleFilter(sepiaCIImage!, aspectRatio:0.5, scale:0.5)
```
![缩小图像](https://ws2.sinaimg.cn/large/006tNc79gy1fzhhsb2jprj305k09n3yp.jpg)

图像的大小可能无法体现（读者可以试验自行查看），图像的宽高比表现很明显。


## 内置过滤器

#### 查询可用的滤镜

我们可以通过 CIFilter 的方法来查询当前系统内置的过滤器种类，当然你也查询官方文档：[Core Image Filter Reference](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/uid/TP40004346)

```
let filterNames = CIFilter.filterNames(inCategory: kCICategoryBuiltIn)
print(filterNames.count) # 当前有 180 个
```

#### 一些值得留意的滤镜

- **模糊** CICategoryBlur
 - CIGaussianBlur : 高斯模糊
 - CIMaskedVariableBlur : 根据蒙版变量模糊
 - CIMotionBlur : 运动模糊，物体移动式模糊
 
- **色彩调节** CICategoryColorAdjustment
 - CIColorClamp : 修改颜色值，使图像中的色值保持在指定范围内
 - CIColorControls : 调整饱和度，亮度和对比度值
 - CIExposureAdjust : 曝光调整
 - CIGammaAdjust : 伽玛调整，中间调亮度
 - CIHueAdjust : 色调调整
 - CITemperatureAndTint : 温度和色调
 - CIVibrance : 调整图像的饱和度
 
- **色彩效果** CICategoryColorEffect
 - CIColorInvert : 颜色反转
 - CIColorMonochrome : 重新映射颜色，使其落入单一颜色的阴影中
 - CIColorPosterize : 将红色，绿色和蓝色组件重新映射为为每个颜色组件指定的亮度值数
 - CIFalseColor : 将亮度映射到两种颜色的颜色渐变
 - CIMaskToAlpha : 将灰度图像转换为由alpha屏蔽的白色图像
 - CIPhotoEffectChrome : 复古摄影胶片，有色彩
 - CIPhotoEffectFade : 复古摄影胶片，褪色
 - CIPhotoEffectInstant : 复古摄影胶片，带有扭曲颜色
 - CIPhotoEffectMono : 黑白摄影胶片，低对比度
 - CIPhotoEffectNoir : 黑白摄影胶片，夸张的对比度
 - CIPhotoEffectProcess : 复古摄影胶片，强调冷色调
 - CIPhotoEffectTonal : 黑白摄影胶片，不会显着改变对比度
 - CIPhotoEffectTransfer : 复古摄影胶片，暖色调
 - CISepiaTone : 棕褐色调
 - CIVignette : 降低周边图像的亮度，边缘渐黑
 - CIVignetteEffect : 修改指定区域周围图像的亮度
 
- **图像组合** CICategoryCompositeOperation 
 - CIAdditionCompositing : 添加颜色成分以达到增亮效果，通常用于添加高光和镜头眩光效果
 - CISourceAtopCompositing : 合成显示背景图像，仅显示源图像中背景可见部分的那些部分
 - CISourceInCompositing : 使用背景图像裁剪输入图像
 - CISourceOutCompositing : 和 CISourceInCompositing 相反，输入图片只保留背景图像未重合的部分
 - CISourceOverCompositing : 将输入图像放在输入背景图像上
 - CISubtractBlendMode : 减去混合模式，从源图像样本颜色中减去背景图像样本颜色

- **失真效果** CICategoryDistortionEffect
 - CIBumpDistortionLinear : 线性拉伸图片
 - CICircularWrap : 围绕透明圆圈包裹图像
 - CIGlassDistortion : 类似玻璃的纹理来扭曲图像，可以使用纹理图像
 - CILightTunnel : 旋转扭曲产生类似隧道效果
 - CIStretchCrop : 通过拉伸或裁剪图像以适合目标尺寸来扭曲图像
 
- **生成器** CICategoryGenerator
 - CIAztecCodeGenerator : 二维码生成器（Aztec）
 - CICheckerboardGenerator : 棋盘图案生成器，黑白方块间隔
 - CICode128BarcodeGenerator : 一维条形码生成器
 - CIConstantColorGenerator : 纯色生成器
 - CIPDF417BarcodeGenerator : 二维条形码生成器
 - CIQRCodeGenerator : 二维条形码生成器
 - CIStripesGenerator : 条纹图案生成器
 
- **几何调节** CICategoryGeometryAdjustment
 - CIAffineTransform : 对图像应用仿射变换
 - CICrop : 裁剪图像
 - CILanczosScaleTransform : 源图像的高质量缩放
 - CIStraightenFilter : 旋转源图像

- **梯度** CICategoryGradient
 - CIGaussianGradient : 高斯梯度，生成从一种颜色到另一种颜色不同的渐变
 - CILinearGradient : 线性梯度
 - CIRadialGradient : 径向渐变梯度

- **边缘效果** CICategorySharpen
 - CISharpenLuminance : 锐化

- **风格化**
 - CIBlendWithAlphaMask : 使用蒙版中的 alpha 值在图像和背景之间进行插值显示
 - CIBloom : 柔化边缘并为图像产生光晕，高光
 - CIComicEffect : 漫画效果
 - CICrystallize : 通过聚合源像素颜色值创建多边形颜色块，结晶化
 - CIDepthOfField : 模拟景深效果
 - CIGloom : 使图像的高光变暗，和 CIBloom 相反
 - CIHexagonalPixellate : 像素化
 - CIHighlightShadowAdjust : 突出显示和阴影
 - CILineOverlay : 用黑色笔触勾勒出图像边缘的草图
 - CIPixellate : 像素化，类似马赛克

 
- **平铺** CICategoryTileEffect 
 - CIAffineTile : 图像应用仿射变换，然后平铺变换后的图像
 - CIKaleidoscope : 万花筒
 
- **过渡转换** CICategoryTransition 
 - CIAccordionFoldTransition : 折叠过渡，从一个图像转换到另一个不同维度的图像
 - CIBarsSwipeTransition : 条形过渡效果
 - CICopyMachineTransition : 复印机效果
 - CIDisintegrateWithMaskTransition : 蒙版定义的形状效果
 - CIDissolveTransition : 溶解效果
 - CIFlashTransition : 通闪光灯效果
 - CIModTransition : 不规则形状的孔效果
 - CIPageCurlTransition : 卷曲页面效果
 - CIPageCurlWithShadowTransition : 卷曲页面效果
 - CIRippleTransition : 波纹效果
 - CISwipeTransition : 滑动效果

----

更多过滤器使用详情和效果请查询官方文档：[Core Image Filter Reference](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/uid/TP40004346)
















