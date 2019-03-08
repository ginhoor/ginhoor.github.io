[官方文档Flexible class](https://docs.flutter.io/flutter/widgets/Flexible-class.html)

# Flexible class

A widget that controls how a child of a [Row](https://docs.flutter.io/flutter/widgets/Row-class.html), [Column](https://docs.flutter.io/flutter/widgets/Column-class.html), or [Flex](https://docs.flutter.io/flutter/widgets/Flex-class.html) flexes.

> 用于控制`Row`、`Column`、`Flex`子控件的控件。



Using a [Flexible](https://docs.flutter.io/flutter/widgets/Flexible-class.html) widget gives a child of a [Row](https://docs.flutter.io/flutter/widgets/Row-class.html), [Column](https://docs.flutter.io/flutter/widgets/Column-class.html), or [Flex](https://docs.flutter.io/flutter/widgets/Flex-class.html) the flexibility to expand to fill the available space in the main axis (e.g., horizontally for a [Row](https://docs.flutter.io/flutter/widgets/Row-class.html) or vertically for a [Column](https://docs.flutter.io/flutter/widgets/Column-class.html)), but, unlike [Expanded](https://docs.flutter.io/flutter/widgets/Expanded-class.html), [Flexible](https://docs.flutter.io/flutter/widgets/Flexible-class.html)does not require the child to fill the available space.

> Flexible控件可以使一个`Row`、`Column`、`Flex`子控件提供扩展的灵活性，以填充主轴上的可用空间（例如，水平方向上的`Row`或者垂直方向上的`Column`）。但是与`Expanded`不同的是，`Flexible`不会要求子控件填充可用空间



A [Flexible](https://docs.flutter.io/flutter/widgets/Flexible-class.html) widget must be a descendant of a [Row](https://docs.flutter.io/flutter/widgets/Row-class.html), [Column](https://docs.flutter.io/flutter/widgets/Column-class.html), or [Flex](https://docs.flutter.io/flutter/widgets/Flex-class.html), and the path from the [Flexible](https://docs.flutter.io/flutter/widgets/Flexible-class.html) widget to its enclosing [Row](https://docs.flutter.io/flutter/widgets/Row-class.html), [Column](https://docs.flutter.io/flutter/widgets/Column-class.html), or [Flex](https://docs.flutter.io/flutter/widgets/Flex-class.html) must contain only [StatelessWidget](https://docs.flutter.io/flutter/widgets/StatelessWidget-class.html)s or [StatefulWidget](https://docs.flutter.io/flutter/widgets/StatefulWidget-class.html)s (not other kinds of widgets, like [RenderObjectWidget](https://docs.flutter.io/flutter/widgets/RenderObjectWidget-class.html)s).

> `Flexible`必须是`Row`、`Column`、`Flex`的子控件，Flexible与装入它的`Row`、`Column`、`Flex`控件之间，只能是`StatelessWidget`或者`StatefulWidget`（不能是别的类型，比如`RenderObjectWidget`）。



See also:

- [Expanded](https://docs.flutter.io/flutter/widgets/Expanded-class.html), which forces the child to expand to fill the available space.

- [Spacer](https://docs.flutter.io/flutter/widgets/Spacer-class.html), a widget that takes up space proportional to it's flex value.

- The [catalog of layout widgets](https://flutter.io/widgets/layout/).

- Inheritance

  [Object](https://docs.flutter.io/flutter/dart-core/Object-class.html)->[Diagnosticable](https://docs.flutter.io/flutter/foundation/Diagnosticable-class.html)-> [DiagnosticableTree](https://docs.flutter.io/flutter/foundation/DiagnosticableTree-class.html) ->[Widget](https://docs.flutter.io/flutter/widgets/Widget-class.html) ->[ProxyWidget](https://docs.flutter.io/flutter/widgets/ProxyWidget-class.html)->[ParentDataWidget](https://docs.flutter.io/flutter/widgets/ParentDataWidget-class.html)[[Flex](https://docs.flutter.io/flutter/widgets/Flex-class.html)]-> Flexible

- Implementers

  [Expanded](https://docs.flutter.io/flutter/widgets/Expanded-class.html)

