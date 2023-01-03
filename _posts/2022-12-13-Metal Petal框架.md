---
layout: post
title: 'Metal Petal框架'
subtitle: '介绍的Metal Petal能力与最佳使用'
categories: 技术
tags: MetalPetal
---

## 前言

[Metal Petal](https://github.com/MetalPetal/MetalPetal)是一个基于[Metal](https://developer.apple.com/metal/)的图像处理框架，旨在通过易于使用的编程界面为静态图像和视频提供实时处理。

## 核心组件

MetalPetal 的一些核心概念与 Apple Core Image 框架中的概念非常相似。

-   MTIContext，对比CIContext

提供渲染`MTIImage`的评估上下文。它还存储了大量缓存和状态信息，因此尽可能重用上下文更高效。

-   MTIImage，对比CIImage

`MTIImage`对象是要处理或生成的图像的表示。它确实直接表示图像位图数据，相反，它拥有生成图像或更准确地说是aMTLTexture所需的所有信息。它由两部分组成，一个是如何生成纹理的配方（`MTIImagePromise`）和其他信息，例如上下文如何缓存图像（`cachePolicy`），以及如何采样纹理（`samplerDescriptor`）。

*   MTIFilter，对比CIFilter

`MTIFilter`表示图像处理效果和控制该效果的任何参数。它产生一个`MTIImage`对象作为输出。要使用过滤器，您可以创建一个过滤器对象，设置其输入图像和参数，然后访问其输出图像。通常，过滤器类拥有静态内核（`MTIKernel`），当您访问其`outputImage`属性时，它会要求具有输入图像和参数的内核生成输出`MTIImage`。

*   MTIKernel

`MTIKernel`表示图像处理例程。`MTIKernel`负责为过滤器创建相应的渲染或计算管道状态，以及为aMTIImage构建`MTIImagePromise`。

## 优化


MetalPetal做了很多优化，它会自动缓存函数、内核状态、采样器状态等。

它利用可编程混合、无内存渲染目标、资源堆和金属性能着色器等金属功能来实现快速高效的渲染。在macOS上，MetalPetal还可以利用苹果芯片的TBDR架构。

在渲染之前，MetalPetal可以查看您的图像渲染图，并**计算出进行渲染所需的最小中间纹理数量**，从而节省内存、能量和时间。

如果可以串联多个“食谱”，以消除冗余的渲染传递，它也可以重新组织图像渲染图。（`MTIContext.isRenderGraphOptimizationEnabled`）

## 并发性考虑因素

`MTIImage`对象是不可变的，这意味着它们可以在线程之间安全地共享。

然而，`MTIFilter`对象是可变的，因此无法在线程之间安全共享。

`MTIContext`包含许多状态和缓存。`MTIContext`对象有一个线程安全机制，使得在线程之间共享`MTIContext`对象是安全的。

## 比 Core Image 的优势

不同：

Core Image可以使用GPU或者CPU渲染。

MetalPetal主要专注于使用GPU进行渲染。



优势：

-   完全可定制的顶点和片段函数。
-   MRT（多个渲染目标）支持。
-   一般来说，性能更好。（需要详细的基准数据）

## 内置过滤器

-   Color Matrix - 颜色矩阵

-   Color Lookup - 颜色查找

    >   Uses an color lookup table to remap the colors in an image.
    >
    >   使用颜色查找表重新映射图像中的颜色。

-   Opacity - 不透明度

-   Exposure - 曝光

-   Saturation - 饱和

-   Brightness - 亮度

-   Contrast - 对比

-   Color Invert - 颜色反转

-   Vibrance - 活力

    >   Adjusts the saturation of an image while keeping pleasing skin tones.调整图像的饱和度，同时保持令人愉悦的肤色。

-   RGB Tone Curve - RGB音调曲线

-   Blend Modes - 混合模式

    * Normal - 标准
    * Multiply - 相乘
    * Overlay - 覆盖
    * Screen - 拍摄
    * Hard Light - 硬光
    * Soft Light - 柔和的光线
    * Darken - 变暗
    * Lighten - 减轻
    * Color Dodge - 彩色道奇
    * Add (Linear Dodge) - 添加（线性道奇）
    * Color Burn - 颜色燃烧
    * Linear Burn - 线性燃烧
    * Lighter Color - 较浅的颜色
    * Darker Color - 深色
    * Vivid Light - 生动的光
    * Linear Light - 线性光
    * Pin Light - 针灯
    * Hard Mix - 硬混合
    * Difference - 区别
    * Exclusion - 排斥
    * Subtract - 减去
    * Divide - 分裂
    * Hue - 顺化
    * Saturation - 饱和
    * Color - 颜色
    * Luminosity - 亮度
    * ColorLookup512x512 - 颜色查找512x512
    * Custom Blend Mode - [自定义混合模式](https://github.com/MetalPetal/MetalPetal/issues/70#issuecomment-792430483)

-   Blend with Mask - 与遮罩混合

-   Transform - 转变

-   Crop - 裁剪

-   Pixellate - 像素

-   Multilayer Composite - 多层复合材料

-   MPS Convolution - MPS卷积

-   MPS Gaussian Blur - MPS高斯模糊

-   MPS Definition - MPS定义

-   MPS Sobel - MPS Sobel

-   MPS Unsharp Mask - MPS不明显的遮罩

-   MPS Box Blur - MPS盒子模糊

-   High Pass Skin Smoothing - [高通皮肤平滑](https://github.com/YuAo/YUCIHighPassSkinSmoothing)

-   CLAHE (Contrast-Limited Adaptive Histogram Equalization) - [CLAHE（对比度限制自适应直方图均衡）](https://github.com/YuAo/Accelerated-CLAHE)

-   Lens Blur (Hexagonal Bokeh Blur) - [镜头模糊（六角形散景模糊）](https://github.com/YuAo/HexagonalBokehBlur)

-   Surface Blur - [表面模糊](https://github.com/MetalPetal/SurfaceBlur)

-   Bulge Distortion - 隆起失真

-   Chroma Key Blend - 色度键混合

-   Color Halftone - 颜色半色调

-   Dot Screen - 点屏

-   Round Corner (Circular/Continuous Curve) - 圆角（圆形/连续曲线）

-   All Core Image Filters - [所有Core Image滤镜](https://github.com/MetalPetal/MetalPetal#working-with-core-image)，配合`MTICoreImageUnaryFilter`使用

## 最佳实践

-   尽可能重复使用`MTIContext`。

上下文是重量级对象，因此，如果您确实创建了对象，请尽早创建，并在每次需要渲染图像时重用它。

-   明智地使用`MTIImage.cachePolicy`。

当您不想保留图像的渲染结果时，请使用`MTIImageCachePolicyTransient`，例如当图像只是过滤器链中的中间结果时，因此渲染结果的底层纹理可以重复使用。这是内存效率最高的选项。但是，当您要求上下文渲染之前渲染的图像时，它可能会重新渲染该图像，因为它的基础纹理已被重用。

默认情况下，过滤器的输出映像具有`transient`策略。

当您想防止底层纹理被重复使用时，请使用`MTIImageCachePolicyPersistent`。

默认情况下，从外部来源创建的图像具有`persistent`策略。

-   了解`MTIFilter.outputImage`是一个计算属性。

每次您要求过滤器输入其输出图像时，即使输入与上次调用相同，过滤器也可能为您提供新的输出图像对象。因此，尽可能重复使用输出图像。

例如，

```
//          ╭→ filterB
// filterA ─┤
//          ╰→ filterC
// 
// filterB and filterC use filterA's output as their input.
```

在这种情况下，以下解决方案：

```swift
let filterOutputImage = filterA.outputImage
filterB.inputImage = filterOutputImage
filterC.inputImage = filterOutputImage
```

比以下更好：

```
filterB.inputImage = filterA.outputImage
filterC.inputImage = filterA.outputImage
```