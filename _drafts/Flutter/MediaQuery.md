## 简介
MediaQuery可以用于查询和解析给定数据的子树。

比如获取当前窗口的大小，可以通过`MediaQuery.of(context).size`。使用`MediaQuery.of`查询当前media会使控件在在MediaQueryData更改时自动重建（例如，如果用户旋转其设备）。

如果MediaQuery没有查到MediaQueryData，将会抛出异常，此时如果想要返回结果为null，需要将`nullOk`参数设置为`true`。

WidgetsApp和MaterialApp，都引入了MediaQuery，并在当前屏幕指标发生变化时使其保持最新。

MediaQueryData表示指标的数据结构。