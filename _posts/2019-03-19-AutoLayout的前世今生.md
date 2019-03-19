---
layout: post
title: 'AutoLayout的前世今生'
subtitle: '今天你用AutoLayout了么？'
categories: 技术
tags: AutoLayout
---

## 诞生

AutoLayout是Apple在iOS 6的时候引入的新特性。

AutoLayout使用算法是Cassowary，Cassowary能够有效解析线性等式和线性不等式，用来表示用户界面中的相等关系和不等关系。同时提供一套规则系统，通过约束来描述View之间的关系。

AutoLayout不止有布局算法，还包含了一套布局引擎系统（Layout Engine），用于维护布局在运行时的生命周期。

## 如何工作

每个View在布局之前，Layout Engine都会先通过计算布局约束，得到View的Size和Position。

Layout Engine会监听约束的变化，当约束发生变化，就会触发约束的重新计算（比如添加、删除View、View的显示隐藏、修改约束条件或者约束优先级）。

在刷新布局时，Layout Engine会从上到下调用layoutSubviews()方法，同时计算出约束的frame，再赋值给View，最后调用superView的setNeedLayout方法来刷新布局。

## 性能问题

问题出现在View嵌套，并且有同层级View相互关联约束的时候（比如ViewA中，有一个ViewB和ViewC，ViewC有一条约束是ViewC的左侧等于ViewB的右侧偏移10个像素），对性能的影响是呈指数型增长的。

iOS 12之前，每次约束变化时，Layout Engine需要创建一个NSISEngier将约束关系重新计算，所以当约束关系多层嵌套后，计算量会呈指数增长。

iOS 12的Auto Layout更多地利用了Cassowary算法的界面更新策略，提高了约束计算的性能（[WWDC 220 Session High Performance Auto Layout](https://developer.apple.com/videos/play/wwdc2018/220)），让AutoLayout可以达到和Frame一样的性能。

## UIStackView

UIStackView就是Apple在iOS 9时推出的新控件，目的就是仿造前端Flexbox的思路，提高开发效率。再两个相关联的View需要排列时，不再需要添加View之间的约束关系，而可以用Fill，leading，Center来约束他们的排列方式。这是一个推荐的用法。



