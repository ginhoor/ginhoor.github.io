[GridView class](https://docs.flutter.io/flutter/widgets/GridView-class.html)

`GridView`是一个可以滚动的、二维数组组成的控件。
`GridView`的滚动方向就是主轴的方向。

#### 继承关系

Object > Diagnosticable > DiagnosticableTree > Widget > StatelessWidget > ScrollView > BoxScrollView > GridView


最常用的布局是`GridView.count`，可以创建固定数量的Item；以及`GridView.extend`，可以创建让Item填充满横轴的布局。一个自定义的`SliverGridDelegate`可以设置一组二维排列的Item，排列方式可以是末端对齐或者重叠。

创建拥有大量Item的Grid时，应当选用`GridView.builder`构造函数，`gridDelegate`可以选择横轴确定数量方案（`SliverGridDelegateWithFixedCrossAxisCount`）或者填充满横轴方案（`SliverGridDelegateWithMaxCrossAxisExtent`）

创建自定义排列方案，可以选用`GridView.custom`，使用（`SliverChildDelegate`）定义排列方案。

通过`controller`的`ScrollController.initialScrollOffset`来定义初始的滑动偏移。

## GridView.count
`GridView.count`适用于单列的横向或者纵向列表布局，超出部分会自动滚动。

```
GridView.count(
  primary: false,
  padding: const EdgeInsets.all(20.0),
  crossAxisSpacing: 10.0,
  crossAxisCount: 2,
  children: <Widget>[
    const Text('He\'d have you all unravel at the'),
    const Text('Heed not the rabble'),
    const Text('Sound of screams but the'),
    const Text('Who scream'),
    const Text('Revolution is coming...'),
    const Text('Revolution, they...'),
  ],
)
```

## GridView.extent

`GridView.extend`，可以规定横/纵轴上的Item最大的像素范围的布局

```
  @override
  Widget build(BuildContext context) {
  
  	 //获取当前环境中的MediaQueryData。
    MediaQueryData data = MediaQuery.of(context);
    
    return GridView.extent(
      maxCrossAxisExtent: data.size.width/2,//将最大横轴宽度设置为屏幕的有一半
      children: _entranceWidgets,
      scrollDirection: Axis.vertical,
    );
  }
```
## 将Grid加入到CustomScrollView中

`GridView`本质上是一个`CustomScrollView`，包含了一个`SliverGrid`在`CustomScrollView.slivers`中。

如果`GridView`不满足需求时，比如滚动视图同时拥有一个网格和列表，或者因为网格需要与AppBar组合在一起，则可以将代码直接移植到`CustomScrollView`中使用。

#### 属性匹配

`GridView`与`CustomScrollView`的共有属性：

1. scrollDirection
2. reverse
3. controller
4. primary
5. physics
6. shrinkwrap

`CustomScrollView.slivers`中应当只包含一个`SliverGrid`。


`GridView`中的`childrenDelegate`属性对应的是`SliverGrid.delegate`；`gridDelegate`对应的是`SliverGrid.gridDelegate`。

`new GridView`、`new GridView.count`和`new GridView.extent`的构造函数中，`childrenDelegate`和`SliverChildListDelegate`中的参数相匹配。

`new GridView.builder`构造函数中，`itemBuilder`和`childCount`对应的`childrenDelegate`也可以匹配`SliverChildBuilderDelegate`相匹配。

`new GridView.count`和`new GridView.extent`的构造函数创建的自定义网格代理（grid delegates），在`SliverGrid`中也有对应的`new SliverGrid.count`和`new SliverGrid.extent`。

`padding`属性将对应`CustomScrollView.slivers`中的`SliverPadding`而不是网格本身，并让`SliverGrid`成为`SliverPadding`的子级。

默认情况下，列表将自动填充到可以滑动的末端。可以使用零边距来关闭这个特性。

`SliverApp`、`SliverList`都可以被放入`CustomScrollView.slivers`中。

