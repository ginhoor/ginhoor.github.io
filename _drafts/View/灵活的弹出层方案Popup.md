# 灵活的弹出层方案Popup

先附上DEMO地址
[DEMO地址](https://github.com/ginhoor/GinPopup)

***
为了解决业务中经常出现的弹层问题，我抽象了一个简单的弹层框架。

这个框架写的很简单，基本看两眼就懂了，也容易扩展。


框架有三部分组成，内容层（View），背景渲染层（View），视图控制器（ViewController）、管理者（简单分装用法）

主要思路是，创建一个自定义window，并作为keyWindow遮盖在当前window之上。在这个window上再绘制需要的背景，内容，动画效果。

这个库进行扩展很简单，只要设计好view，然后放入ViewController展示就可以，创建一个GinPopup的category就可以适应不同的view了。