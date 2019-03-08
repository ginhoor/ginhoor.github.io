[官方文档ConstrainedBox class](https://docs.flutter.io/flutter/widgets/ConstrainedBox-class.html)

# ConstrainedBox class



A widget that imposes additional constraints on its child.

> ConstrainedBox 可以为它的子控件强制加上一个约束条件。



For example, if you wanted [child](https://docs.flutter.io/flutter/widgets/SingleChildRenderObjectWidget/child.html) to have a minimum height of 50.0 logical pixels, you could use `const BoxConstraints(minHeight: 50.0)` as the [constraints](https://docs.flutter.io/flutter/widgets/ConstrainedBox/constraints.html).

> 比如你希望子控件拥有最小50像素的高度，你就可以使用`const BoxConstraints(minHeight: 50.0)`



Sample

This snippet makes the child widget (a [Card](https://docs.flutter.io/flutter/material/Card-class.html) with some [Text](https://docs.flutter.io/flutter/widgets/Text-class.html)) fill the parent, by applying [BoxConstraints.expand](https://docs.flutter.io/flutter/rendering/BoxConstraints/BoxConstraints.expand.html) constraints:

```dart
ConstrainedBox(
  constraints: const BoxConstraints.expand(),
  child: const Card(child: Text('Hello World!')),
)
```



The same behavior can be obtained using the [new SizedBox.expand](https://docs.flutter.io/flutter/widgets/SizedBox/SizedBox.expand.html) widget.

See also:

- [BoxConstraints](https://docs.flutter.io/flutter/rendering/BoxConstraints-class.html), the class that describes constraints.
- [UnconstrainedBox](https://docs.flutter.io/flutter/widgets/UnconstrainedBox-class.html), a container that tries to let its child draw without constraints.
- [SizedBox](https://docs.flutter.io/flutter/widgets/SizedBox-class.html), which lets you specify tight constraints by explicitly specifying the height or width.
- [FractionallySizedBox](https://docs.flutter.io/flutter/widgets/FractionallySizedBox-class.html), which sizes its child based on a fraction of its own size and positions the child according to an [Alignment](https://docs.flutter.io/flutter/painting/Alignment-class.html) value.
- [AspectRatio](https://docs.flutter.io/flutter/widgets/AspectRatio-class.html), a widget that attempts to fit within the parent's constraints while also sizing its child to match a given aspect ratio.
- The [catalog of layout widgets](https://flutter.io/widgets/layout/).

- Inheritance

  [Object](https://docs.flutter.io/flutter/dart-core/Object-class.html)-> [Diagnosticable](https://docs.flutter.io/flutter/foundation/Diagnosticable-class.html)-> [DiagnosticableTree](https://docs.flutter.io/flutter/foundation/DiagnosticableTree-class.html)-> [Widget](https://docs.flutter.io/flutter/widgets/Widget-class.html)-> [RenderObjectWidget](https://docs.flutter.io/flutter/widgets/RenderObjectWidget-class.html)-> [SingleChildRenderObjectWidget](https://docs.flutter.io/flutter/widgets/SingleChildRenderObjectWidget-class.html)-> ConstrainedBox

