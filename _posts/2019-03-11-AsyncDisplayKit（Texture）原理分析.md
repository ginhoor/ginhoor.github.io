---
layout: post
title: 'AsyncDisplayKit（Texture）原理分析'
categories: 技术
tags: AsyncDisplayKit Texture 异步渲染
---

## ASDK涉及的UI任务
Layout

1. 文本宽高计算
2. 视图布局计算

Rendering

1. 文本渲染
2. 图片解码
3. 图形绘制

UIKit Objects

1. 对象创建
2. 对象调整
3. 对象销毁

ASDK尝试将这些任务放到异步线程处理，如UIKit与CoreAnimation只能在主线程操作的，则进行优化。

## ASDisplayNode

UIView与CALayer只能在主线程创建和销毁，所以ASDK创建了ASDisplayNode作为中间类来处理UI关系。当ASDisplayNode不响应触摸时间时，它将只作为CALayer处理。

## ASDK的图片预合成

一个Node中，往往包含多个层级的Node组合。当这些Node不需要响应触摸时间，也不需要进行动画和位置变化时，ASDK可以开启预合成功能。

将Node都设置为layer backed后，ASDK会将这些Node合成并渲染成一张图，避免CPU创建UIKit对象的资源消耗，避免GPU多张texture合成和渲染的消耗。

## ASDK的多线程

ASK会将布局计算、文本排版、图片/文本/图形的渲染封装成较小的任务，然后用GCD来异步并发执行。

## RunLoop 任务分发

#### 信号传递

1.硬件 -（VSync）-> 2.图形服务 -（IPC）-> 3.RunLoop -> 4.SourceObserver(CoreAnimation、CADisplayLink、AutoreleasePool等)

系统作用范围：2，3，4。
App作用范围：3，4。

iOS的显示是由VSync信号驱动的，VSync信号由硬件时钟生成的，每秒发出60次。iOS图形服务收到VSync信号后，会通过IPC通知到App中。App的RunLoop启动后会注册对应的CFRunLoopSource，通过mach_port接受传过来的时钟信号，随后Source的回调会驱动App的动画和显示。

#### Core Animation

1. Core Animation在RunLoop中注册了一个Observer，监听BeforeWaiting和Exit事件。这个Observer的优先级是2000000，低于常见的其他ObServer。

2. 当一个触摸事件到来时，RunLoop被唤醒，App的代码开始执行操作，比如创建和调整View层级、设置View的Frame、修改CALayer透明度、为视图添加一个动画等；

3. 这些操作会被CALayer捕获，通过CATransacation提交到一个中间状态上。

4. 当上面所有操作结束后，RunLoop即将进入休眠或退出时，关注该事件的Observer都会接收到通知。这是Core Animation注册的Observer就会在回调中，将中间状态合并提交到GPU进行渲染，如果此处有动画，Core Animation会通过CADisplayLink等机制多次触发相关流程。

#### ASDK的机制
ASDK就是模拟了Core Animation的机制：当对ASNode的修改和提交涉及到必须主线程运行的函数时，ASNode会将任务用ASAsyncTransaction（Group）封装并提交到一个全局容器中。ASDK也在RunLoop中注册了一个Observer，监听的事件与Core Animation相同（都是BeforWaiting和Exit事件），优先级设置的比Core Animation低。当RunLoop进入休眠前，Core Animation事件处理完毕后，ASDK会执行提交在该Loop的任务。

通过这种机制，ASDK可以合理的把异步、并发操作同步到主线程中，获得不错的性能。

## 参考资料

[iOS 保持界面流畅的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)