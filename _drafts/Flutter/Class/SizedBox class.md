[官方文档SizedBox class](https://docs.flutter.io/flutter/widgets/SizedBox-class.html)

# SizedBox class

A box with a specified size.

> 具有指定大小的盒子



If given a child, this widget forces its child to have a specific width and/or height (assuming values are permitted by this widget's parent). If either the width or height is null, this widget will size itself to match the child's size in that dimension.

> 如果赋予子控件，Sizedbox会强制它的子控件有指定的宽度和（或者）高度（赋予的值必须是SizedBox的父控件允许的）。如果宽度或者高度为null，SizedBox会在这个维度中将它自己的大小调整为子控件的大小。



If not given a child, this widget will size itself to the given width and height, treating nulls as zero.

> 如果没有赋予子控件，SizedBox会将会把自己的大小调整给定大小，null即为size zero。



The [new SizedBox.expand](https://docs.flutter.io/flutter/widgets/SizedBox/SizedBox.expand.html) constructor can be used to make a [SizedBox](https://docs.flutter.io/flutter/widgets/SizedBox-class.html) that sizes itself to fit the parent. It is equivalent to setting [width](https://docs.flutter.io/flutter/widgets/SizedBox/width.html) and [height](https://docs.flutter.io/flutter/widgets/SizedBox/height.html) to [double.infinity](https://docs.flutter.io/flutter/dart-core/double/infinity-constant.html).

> 使用 new SizedBox.expand 初始化函数可以创建一个自适应父控件的的SizedBox。它的宽和高相当于被设置为double.infinity（无穷大）。



Sample

This snippet makes the child widget (a [Card](https://docs.flutter.io/flutter/material/Card-class.html) with some [Text](https://docs.flutter.io/flutter/widgets/Text-class.html)) have the exact size 200x300, parental constraints permitting:

```dart
SizedBox(
  width: 200.0,
  height: 300.0,
  child: const Card(child: Text('Hello World!')),
)
```



See also:

- [ConstrainedBox](https://docs.flutter.io/flutter/widgets/ConstrainedBox-class.html), a more generic version of this class that takes arbitrary [BoxConstraints](https://docs.flutter.io/flutter/rendering/BoxConstraints-class.html) instead of an explicit width and height.
- [UnconstrainedBox](https://docs.flutter.io/flutter/widgets/UnconstrainedBox-class.html), a container that tries to let its child draw without constraints.
- [FractionallySizedBox](https://docs.flutter.io/flutter/widgets/FractionallySizedBox-class.html), a widget that sizes its child to a fraction of the total available space.
- [AspectRatio](https://docs.flutter.io/flutter/widgets/AspectRatio-class.html), a widget that attempts to fit within the parent's constraints while also sizing its child to match a given aspect ratio.
- [FittedBox](https://docs.flutter.io/flutter/widgets/FittedBox-class.html), which sizes and positions its child widget to fit the parent according to a given [BoxFit](https://docs.flutter.io/flutter/painting/BoxFit-class.html)discipline.
- The [catalog of layout widgets](https://flutter.io/widgets/layout/).

- Inheritance

  [Object](https://docs.flutter.io/flutter/dart-core/Object-class.html)> [Diagnosticable](https://docs.flutter.io/flutter/foundation/Diagnosticable-class.html)> [DiagnosticableTree](https://docs.flutter.io/flutter/foundation/DiagnosticableTree-class.html)> [Widget](https://docs.flutter.io/flutter/widgets/Widget-class.html)> [RenderObjectWidget](https://docs.flutter.io/flutter/widgets/RenderObjectWidget-class.html)> [SingleChildRenderObjectWidget](https://docs.flutter.io/flutter/widgets/SingleChildRenderObjectWidget-class.html)> SizedBox