[官方文档Positioned class](https://docs.flutter.io/flutter/widgets/Positioned-class.html)

# Positioned class

A widget that controls where a child of a [Stack](https://docs.flutter.io/flutter/widgets/Stack-class.html) is positioned.

> 一个用来控制它的子控件在Stack中的放置位置。



A [Positioned](https://docs.flutter.io/flutter/widgets/Positioned-class.html) widget must be a descendant of a [Stack](https://docs.flutter.io/flutter/widgets/Stack-class.html), and the path from the [Positioned](https://docs.flutter.io/flutter/widgets/Positioned-class.html) widget to its enclosing [Stack](https://docs.flutter.io/flutter/widgets/Stack-class.html) must contain only [StatelessWidget](https://docs.flutter.io/flutter/widgets/StatelessWidget-class.html)s or [StatefulWidget](https://docs.flutter.io/flutter/widgets/StatefulWidget-class.html)s (not other kinds of widgets, like[RenderObjectWidget](https://docs.flutter.io/flutter/widgets/RenderObjectWidget-class.html)s).

> 一个`Positiond`控件必须是`Stack`的子控件，在`Positiond`控件与装入它的`Stack`之间，必须只包含`StatelessWidget`或者`StatefulWidget`。（不能是别的类型，比如RenderObjectWidget）。



If a widget is wrapped in a [Positioned](https://docs.flutter.io/flutter/widgets/Positioned-class.html), then it is a *positioned* widget in its [Stack](https://docs.flutter.io/flutter/widgets/Stack-class.html). If the [top](https://docs.flutter.io/flutter/widgets/Positioned/top.html) property is non-null, the top edge of this child will be positioned [top](https://docs.flutter.io/flutter/widgets/Positioned/top.html) layout units from the top of the stack widget. The [right](https://docs.flutter.io/flutter/widgets/Positioned/right.html), [bottom](https://docs.flutter.io/flutter/widgets/Positioned/bottom.html), and [left](https://docs.flutter.io/flutter/widgets/Positioned/left.html) properties work analogously.

> 如果一个控件被`Positioned`包裹，那么它在`Stack`中就是一个已经定位的控件。如果top属性不为空，则这个`widget`的顶部将与`Stack`的顶部间隔与`top`值相等单位的距离。右侧，底部和左侧属性的工作方式类似。



If both the [top](https://docs.flutter.io/flutter/widgets/Positioned/top.html) and [bottom](https://docs.flutter.io/flutter/widgets/Positioned/bottom.html) properties are non-null, then the child will be forced to have exactly the height required to satisfy both constraints. Similarly, setting the [right](https://docs.flutter.io/flutter/widgets/Positioned/right.html) and [left](https://docs.flutter.io/flutter/widgets/Positioned/left.html) properties to non-null values will force the child to have a particular width. Alternatively the [width](https://docs.flutter.io/flutter/widgets/Positioned/width.html) and [height](https://docs.flutter.io/flutter/widgets/Positioned/height.html) properties can be used to give the dimensions, with one corresponding position property (e.g. [top](https://docs.flutter.io/flutter/widgets/Positioned/top.html) and [height](https://docs.flutter.io/flutter/widgets/Positioned/height.html)).

> 如果`top`和`bottom`属性都不为空，则控件将被赋予满足这两项约束的高度。相同的，设置`left`和`right`属性，将为控件赋予满足约束的宽度。或者给出宽度或者高度属性，加上一个相应的位置属性，来声明控件尺寸（比如，`top`和`height`）。



If all three values on a particular axis are null, then the [Stack.alignment](https://docs.flutter.io/flutter/widgets/Stack/alignment.html) property is used to position the child.

> 如果三个特定轴上的值都为空，则`Stack.alignment`属性将用来为子控件排序。



If all six values are null, the child is a non-positioned child. The [Stack](https://docs.flutter.io/flutter/widgets/Stack-class.html) uses only the non-positioned children to size itself.

> 如果所有六个值都为空，则子控件是一个未定位的控件。`Stack`将会让未定位控件适应它的大小。



See also:

- [PositionedDirectional](https://docs.flutter.io/flutter/widgets/PositionedDirectional-class.html), which adapts to the ambient [Directionality](https://docs.flutter.io/flutter/widgets/Directionality-class.html).

- Inheritance

  [Object](https://docs.flutter.io/flutter/dart-core/Object-class.html)> [Diagnosticable](https://docs.flutter.io/flutter/foundation/Diagnosticable-class.html)> [DiagnosticableTree](https://docs.flutter.io/flutter/foundation/DiagnosticableTree-class.html)> [Widget](https://docs.flutter.io/flutter/widgets/Widget-class.html)> [ProxyWidget](https://docs.flutter.io/flutter/widgets/ProxyWidget-class.html)> [ParentDataWidget](https://docs.flutter.io/flutter/widgets/ParentDataWidget-class.html)[[Stack](https://docs.flutter.io/flutter/widgets/Stack-class.html)] >Positioned