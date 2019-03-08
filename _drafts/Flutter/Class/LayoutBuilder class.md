[官方文档LayoutBuilder class](https://docs.flutter.io/flutter/widgets/LayoutBuilder-class.html)

# LayoutBuilder class

Builds a widget tree that can depend on the parent widget's size.

> 构建依赖于父控件大小的控件树



Similar to the [Builder](https://docs.flutter.io/flutter/widgets/Builder-class.html) widget except that the framework calls the [builder](https://docs.flutter.io/flutter/widgets/LayoutBuilder/builder.html) function at layout time and provides the parent widget's constraints. This is useful when the parent constrains the child's size and doesn't depend on the child's intrinsic size. The [LayoutBuilder](https://docs.flutter.io/flutter/widgets/LayoutBuilder-class.html)'s final size will match its child's size.

> 与创建控件相似，除了框架在布局时候调用`builder`并提供父控件的约束信息。适合需要父控件约束子控件大小，而不是依赖子控件的内在大小的时候使用。最终`LayoutBuilder`的大小会匹配它的子控件大小。



If the child should be smaller than the parent, consider wrapping the child in an [Align](https://docs.flutter.io/flutter/widgets/Align-class.html) widget. If the child might want to be bigger, consider wrapping it in a [SingleChildScrollView](https://docs.flutter.io/flutter/widgets/SingleChildScrollView-class.html).

> 如果子控件需要小于父控件，可以考虑将子控件放在带有排序功能的控件中。如果子控件想要变得更大，可以考虑放在`SingleChildScrollView`中。



See also:

- [Builder](https://docs.flutter.io/flutter/widgets/Builder-class.html), which calls a `builder` function at build time.
- [StatefulBuilder](https://docs.flutter.io/flutter/widgets/StatefulBuilder-class.html), which passes its `builder` function a `setState` callback.
- [CustomSingleChildLayout](https://docs.flutter.io/flutter/widgets/CustomSingleChildLayout-class.html), which positions its child during layout.

- Inheritance

  [Object](https://docs.flutter.io/flutter/dart-core/Object-class.html) >[Diagnosticable](https://docs.flutter.io/flutter/foundation/Diagnosticable-class.html) >[DiagnosticableTree](https://docs.flutter.io/flutter/foundation/DiagnosticableTree-class.html) >[Widget](https://docs.flutter.io/flutter/widgets/Widget-class.html)> [RenderObjectWidget](https://docs.flutter.io/flutter/widgets/RenderObjectWidget-class.html)> LayoutBuilder