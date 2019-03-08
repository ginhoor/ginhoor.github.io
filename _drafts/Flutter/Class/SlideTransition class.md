[官方文档SlideTransition class](https://docs.flutter.io/flutter/widgets/SlideTransition-class.html)

# SlideTransition class



Animates the position of a widget relative to its normal position.

> 相对于控件的正常位置进行位置动画。



The translation is expressed as a [Offset](https://docs.flutter.io/flutter/dart-ui/Offset-class.html) scaled to the child's size. For example, an [Offset](https://docs.flutter.io/flutter/dart-ui/Offset-class.html) with a `dx` of 0.25 will result in a horizontal translation of one quarter the width of the child.

> 这个转化用偏移量表示，偏移量与子控件大小成比例。例如，一个关于dx的偏移量为0.25，将导致一个子控件宽度四分之一的水平平移。



By default, the offsets are applied in the coordinate system of the canvas (so positive x offsets move the child towards the right). If a [textDirection](https://docs.flutter.io/flutter/widgets/SlideTransition/textDirection.html) is provided, then the offsets are applied in the reading direction, so in right-to-left text, positive x offsets move towards the left, and in left-to-right text, positive x offsets move towards the right.

> 默认情况下，偏移量用的是画布的坐标体系（比如为正值偏移量，将导致子控件向右移动）。如果提供了`textDirection`则偏移量会按照读取方向运行，比如在从右往左的Text中，正值偏移量将导致往左移动，从左到右的Text中，正值偏移量将导致向右移动。




Here's an illustration of the [SlideTransition](https://docs.flutter.io/flutter/widgets/SlideTransition-class.html) widget, with it's [position](https://docs.flutter.io/flutter/widgets/SlideTransition/position.html) animated by a [CurvedAnimation](https://docs.flutter.io/flutter/animation/CurvedAnimation-class.html) set to [Curves.elasticIn](https://docs.flutter.io/flutter/animation/Curves/elasticIn-constant.html):

<video id="animation_1" loop="" style="box-sizing: inherit; width: 300px; height: 378px;"></video>

See also:

- [AlignTransition](https://docs.flutter.io/flutter/widgets/AlignTransition-class.html), an animated version of an [Align](https://docs.flutter.io/flutter/widgets/Align-class.html) that animates its [Align.alignment](https://docs.flutter.io/flutter/widgets/Align/alignment.html) property.
- [PositionedTransition](https://docs.flutter.io/flutter/widgets/PositionedTransition-class.html), a widget that animates its child from a start position to an end position over the lifetime of the animation.
- [RelativePositionedTransition](https://docs.flutter.io/flutter/widgets/RelativePositionedTransition-class.html), a widget that transitions its child's position based on the value of a rectangle relative to a bounding box.

- Inheritance

  [Object](https://docs.flutter.io/flutter/dart-core/Object-class.html)> [Diagnosticable](https://docs.flutter.io/flutter/foundation/Diagnosticable-class.html)> [DiagnosticableTree](https://docs.flutter.io/flutter/foundation/DiagnosticableTree-class.html)> [Widget](https://docs.flutter.io/flutter/widgets/Widget-class.html)> [StatefulWidget](https://docs.flutter.io/flutter/widgets/StatefulWidget-class.html)> [AnimatedWidget](https://docs.flutter.io/flutter/widgets/AnimatedWidget-class.html)> SlideTransition