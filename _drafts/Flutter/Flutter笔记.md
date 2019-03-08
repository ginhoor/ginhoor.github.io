1. Material 是一个视觉设计语言。是移动端和网页端的标准。Flutter提供丰富的Material组件库。
2. Flutter中几乎所有的东西都是控件，包括对齐，间距，布局
3. 一个控件的主要作用就是提供build()方法，用来描述如何显示。

pubspec.yaml：负责依赖引用的配置

StatelessWidget是不可变的，意味着里面所有的属性都是final类型。

Stateful widgets 在widget生命周期中，维护状态可以改变。

创造一个StatefulWidget至少要满足两点
1. 一个StatefulWodget类，可以用来创建实例。
2. 一个State类

StatefulWidget本身是不能变的，但是State在插件的生命周期中持久存在。