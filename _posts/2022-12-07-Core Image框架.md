---
layout: post
title: 'Core Image框架'
subtitle: '解读官方文档中Core Image的能力与最佳使用'
categories: 技术
tags: CoreImage
---

## 前言

Core Image是一种图像处理和分析的技术，用于对静态图像和视频图像提供近似实时的处理。使用GPU或者CPU渲染，支持来自Core Graphics、Core Video和Image I/O的图像数据类型进行操作。

Core Image通过提供易于使用的API来隐藏底层图形处理的细节。

### 能力

*   提供内置的图像滤镜
*   特征检测能力，如人脸检测
*   支持自动的图像增强，如修复红眼，平衡肤色，调整对比度与阴影
*   将多个图像滤镜组合在一起，创造自定义的效果
*   支持创建运行在GPU上的自定义滤镜
*   给予反馈的图像处理能力
*   实时的视频图像处理

### 常用场景

*   实现图像的艺术效果，例如风格化、半色调滤镜等。也适合修复图像问题，如颜色调整和锐化滤镜。
*   用于分析图像质量，如调整色调，对比度，色调颜色等，以及纠正红颜等闪光灯伪影。
*   检测静态图像中的人脸特征，并随着时间推移，在视频图像中跟踪它们。

>   不要使用Core Image来创建模糊效果，而是使用`NSVisualEffectView`（macOS）或`UIVisualEffectView`（iOS/tvOS）类，它们会自动匹配系统外观并提供高效的实时渲染。

## 图像处理

依赖于CIFilter和CIImage类，它们描述了滤镜的输入与输出。

### 处理流程

```swift
import CoreImage
let context = CIContext()                                           // 1
let filter = CIFilter(name: "CISepiaTone")!                         // 2
filter.setValue(0.8, forKey: kCIInputIntensityKey)
let image = CIImage(contentsOfURL: myURL)                           // 3
filter.setValue(image, forKey: kCIInputImageKey)
let result = filter.outputImage!                                    // 4
let cgImage = context.createCGImage(result, from: result.extent)    // 5
```

1.   创建CIContext对象。我们并不总是需要创建新的CIContext，通常会从其他管理渲染的框架中获取。创建自定义的上下文可以更精准的控制渲染过程以及渲染所需要的资源。Context是高资源消耗的对象，使用时尽早创建，并且在处理图像时进行复用。
2.   创建需要使用的滤镜，并配置参数。
3.   创建一个表示待处理图像的CIImage对象，并为其作为滤镜的输入图像。
4.   获得表示滤镜输出结果的CIImage对象。**此时滤镜尚未实际生效，Core Image会在请求渲染时执行该对象的生成。**
5.   将输出图像渲染成Core Graphics图像，即CGImage，这样我们就可以显示图像或者保存成文件。

### CIImage对象

CIImage对象表示的是一个不可变的待渲染图像信息集合，并不直接表示图像的位图信息。

-   作为输入图像，它会描述如何加载图像数据

-   作为输出图像，它会描述执行滤镜的步骤，即如何从滤镜处理中输出图像。只有在执行渲染请求时，Core Image才会通过CIImage去生成位图数据。

#### 输入图像

*   通过图像文件的URL
*   通过图像的NSData二进制数据
*   通过CGImageRef(Quartz2D)、UIImage(UIKit) 或者 NSBitmapImageRep(AppKit)
*   通过Metal、OpenGL 或者OpenGL ES的纹理
*   通过Core Video的 CVImageBufferRef(图像)  或者 CVPixcelBufferRef(像素缓存数据)
*   通过IOSurfaceRef在进程之间共享的图像数据。
*   通过内存中的图像内存数据（如指向此类数据的指针，或者CIImageProvider对象）

有关创建`CIImage`对象的完整列表，请参阅*[CIimage类参考](https://developer.apple.com/documentation/coreimage/ciimage)*。

####输出图像

作为CIFilter的outputImage属性的类型。

可以通过CIContext或者draw方法来显式请求渲染（请参阅[使用Core Image Context构建自己的工作流程](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/CoreImaging/ci_tasks/ci_tasks.html#//apple_ref/doc/uid/TP30001185-CH3-SW5)）

可以通过使用其他框架来实现隐式请求渲染（请参阅[与其他框架集成](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/CoreImaging/ci_tasks/ci_tasks.html#//apple_ref/doc/uid/TP30001185-CH3-SW6)）

### CIFilter

表示图像的处理效果，以及控制该效果的参数数值。需要指定输入参数，然后从outputImage中获得处理结果。

系统内集成很多滤镜，我们可以通过滤镜的名称来获取使用（请参阅[查询系统以获取滤镜](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/CoreImaging/ci_concepts/ci_concepts.html#//apple_ref/doc/uid/TP30001185-CH2-TPXREF101)或*[Core Image滤镜参考](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/uid/TP40004346)*）

## 线程安全

CIContext 和 CIImage 对象是不可变的，在线程之间共享这些对象是安全的。所以多个线程可以使用同一个 GPU 或者 CPU CIContext 对象来渲染 CIImage 对象。

## 多个滤镜的链式调用

每个Core Image的滤镜都会生成一个输出CIImage对象，因此可以使用该对象作为另一个滤镜的输入，来达到链式调用的目的。

Core Image会优化向这样的链式吊用，以快速有效的渲染结果。链中的每个CIImage对象都不会完整渲染成图像，而是传递滤镜的处理步骤和参数。Core Image不会单独执行每个滤镜，也不会在过程中产生临时像素缓存区。Core Image会重新组织滤镜，甚至可能重新排列滤镜的应用顺序（如Color -> Bloom -> Crop，会被优化为 Crop -> Color -> Bloom），来更有效率的渲染结果图。

```swift
func applyFilterChain(to image: CIImage) -> CIImage {
    // The CIPhotoEffectInstant filter takes only an input image
    let colorFilter = CIFilter(name: "CIPhotoEffectProcess", withInputParameters:
        [kCIInputImageKey: image])!
    
    // Pass the result of the color filter into the Bloom filter
    // and set its parameters for a glowy effect.
    let bloomImage = colorFilter.outputImage!.applyingFilter("CIBloom",
                                                             withInputParameters: [
                                                                kCIInputRadiusKey: 10.0,
                                                                kCIInputIntensityKey: 1.0
        ])
    
    // imageByCroppingToRect is a convenience method for
    // creating the CICrop filter and accessing its outputImage.
    let cropRect = CGRect(x: 350, y: 350, width: 150, height: 150)
    let croppedImage = bloomImage.cropping(to: cropRect)
    
    return croppedImage
}
```

## 特殊滤镜

大多数内置的Core Image滤镜是在输入图的基础上处理，生成单个输出图像。但还有几种其他类型的滤镜，可以与常用的滤镜组合，创建有趣的效果，但这也需要更复杂的工作流程。

### Compositing / Blending

使用预设的公式组合两张照片。

*   [CISourceInCompositing](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CISourceInCompositing)滤镜在合成两张图像后，输出两张图像中不透明的区域。
*   [CIMultiplyBlendMode](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CIMultiplyBlendMode)滤镜在合成两张图像后，将两张图像的像素颜色相乘，产生变暗的输出图像。

更多有关Compositing / Blending滤镜的完整列表，请查询[CICategoryCompositeOperation](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/uid/TP30000136-SW71)。

**注意：**在合成输入图像前，可以对图像进行几何调整。请参阅[CICategoryGeometryAdjustment](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/uid/TP30000136-SW157)滤镜或`imageByApplyingTransform:`方法。

### Generator

不需要输入图像，这些滤镜会根据输入参数，创建新的图像。

一部分Generator滤镜可以直接产出可供使用的结果，还有一部分可以应用在滤镜的链式调用中，组合出更多的结果。

*   [CIQRCodeGenerator](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CIQRCodeGenerator)和[CICode128BarcodeGenerator](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CICode128BarcodeGenerator)等滤镜可以通过输入的数据生成条形码图像
*   [CIConstantColorGenerator](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CIConstantColorGenerator)、[CICheckerboardGenerator](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CICheckerboardGenerator)和[CILinearGradient](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CILinearGradient)等滤镜可以通过指定颜色生成简单的图像。可以将这些与常用滤镜结合使用——例如，[CIRadialGradient](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CIRadialGradient)滤镜可以创建一个与[CIMaskedVariableBlur](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CIMaskedVariableBlur)滤镜一起使用的遮罩。
*   [CILenticularHaloGenerator](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CILenticularHaloGenerator)和[CISunbeamsGenerator](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CISunbeamsGenerator)等滤镜可以创建独立的视觉效果——将这些效果与合成滤镜结合使用，可以为图像添加特殊效果。

更多Generator滤镜，请查询[CICategoryGenerator](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/uid/TP30000136-SW142)和[CICategoryGradient](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/uid/TP30000136-SW166)类别。

### Reduction

其输出的不是创建位图的信息，而是描述输入图像的信息。比如：

-   [CIAreaMaximum](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CIAreaMaximum)滤镜输出的是单个颜色值，代表图像指定区域中所有像素颜色中最亮的颜色。
-   [CIAreaHistogram](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CIAreaHistogram)滤镜输出的是有关图像指定区域中每个强度值的像素数的信息。

所有Core Image 滤镜都必须产生一个`CIImage`对象作为输出，因此Reduction滤镜生成的信息仍然是图像。但通常不会显示这些图像，而是从单像素或单行图像中读取颜色值，或将其用作其他滤镜的输入图像。

有关Reduction滤镜的完整列表，请查询[CICategoryReduction](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/uid/TP30000136-SW184)类别。

### Transition

接收两个输入图像，并根据某个变量生成图像之间的变化。

通常该变量是时间，因此您可以使用Transition滤镜创建动画，该动画以一张图像开始，以另一张图像结束，并使用有趣的视觉效果从一个图像进行到另一个图像。Core Image 提供了几个内置的过渡滤镜，包括：

-   [CIDissolveTransition](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CIDissolveTransition)滤镜可以产生一个简单的交叉溶解，从一个图像流向另一个图像。
-   [CICopyMachineTransition](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/filter/ci/CICopyMachineTransition)滤镜可以模拟复印机，在一张图像上滑动一条亮光条以显示另一张图像。

有关Transition滤镜的完整列表，请查询[CICategoryTransition](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Reference/CoreImageFilterReference/index.html#//apple_ref/doc/uid/TP30000136-SW239)类别。

### 通过Quartz 2D基于CPU进行渲染

如果App对实时性能要求不高，可以通过Core Graphics来做视图渲染(例如在`drawRect:`中)，可以使用 `contextWithCGContext:options:`通过Core Graphics Context来获取CIContext。

## 最佳实践

### 前期准备

在做性能优化前，先判断当前业务的情况。可以从以下几方面进行分析

*   Core Image的执行频率如何
*   是用于静态图像，还是视频图像
*   是否需要支持实时处理或者分析
*   业务对色彩的保真度要求如何

### 优化

在确定了业务情况后，在着手性能优化。有以下几点建议。

*   不要每次渲染时都创建CIContext对象，改对象存储了大量的状态信息，复用它们将提高效率。

*   如果不需要颜色管理，则不要开启它。要禁用颜色管理，请将`kCIImageColorSpace`键设置为`null`。

    *   默认情况下，Core Image 会将所有滤镜应用到光线性色彩空间中。这提供了最准确和一致的结果。转换到sRGB增加了滤镜的复杂性，并要求Core Image应用以下方程：

          ```swif
          rgb = mix(rgb.0.0774, pow(rgb*0.9479 + 0.05213, 2.4), step(0.04045, rgb))
          rgb = mix(rgb12.92, pow(rgb*0.4167) * 1.055 - 0.055, step(0.00313, rgb))  
        ```
        
    *   在以下情形中，请考虑停用色彩管理：
        *   您的应用程序需要绝对最高的性能。
        *   用户在夸张操作后不会注意到质量差异。

*   在GPU上使用CIImage对象时，避免使用Core Animation动画。如果要同时使用，可以将两者都设置为使用CPU

*   确保图像不会超过CPU和GPU的限制。

    *   `CIContext`对图像大小的限制会因Core Image使用CPU还是GPU有所不同。
    *   使用方法`inputImageMaximumSize`和`outputImageMaximumSize`检查限制。

*   使用更小的图像。性能与输出像素数量成反比，像素越多，性能越差。

    *   您可以将 Core Image 渲染到较小的视图、纹理或帧缓冲区。再通过允许Core Animation缩放到展示尺寸。
    *   使用Core Graphics或Image I/O函数 裁剪 或 缩小取样，例如`CGImageCreateWithImageInRect`或`CGImageSourceCreateThumbnailAtIndex`函数。

*   UIImageView类最适合展示静态图像。如果你想要更高的性能，可以可以使用更底层的API

*   避免在CPU和GUP之间进行不必要的纹理传输。

*   渲染与源图相同大小的矩形，而不是用缩放因子（sacle factor）去控制。

*   考虑使用更简单的滤镜，用简单的滤镜去制作近似算法滤镜的结果。例如，CIColorCube可以产生类似于CISepiaTone的输出，并且执行得更有效。

*   利用iOS 6.0及更高版本中对YUV图像的支持。相机像素缓冲区原生是YUV，但大多数图像处理算法都期望RBGA数据。在两者之间转换是有成本的。Core Image支持从`CVPixelBuffer`对象读取YUB并应用适当的颜色转换。（`options = @{ (id)kCVPixelBufferPixelFormatTypeKey :@(kCVPixelFormatType_420YpCbCr88iPlanarFullRange) };`）

## 参考资料

[官方文档](https://developer.apple.com/library/archive/documentation/GraphicsImaging/Conceptual/CoreImaging/ci_intro/ci_intro.html#//apple_ref/doc/uid/TP30001185)