[深入了解Flutter界面开发](https://mp.weixin.qq.com/s/z2r2OmnY7r7dQrkO8ndkFQ)

## 架构
1. Flutter使用的是Skia高性能渲染引擎绘制，没有使用WebView或者系统控件。
2. 界面语言使用dart开发，底层渲染使用的是C、C++。
3. 组合大于继承，控件本身通常由去多小型、单用途的空间组成。类的层次结构是扁平的，以最大化可能的组合数量。

## 渲染路径（Rendering Pipline）
GPU > Animate > Build > Layout > Paint > Layer Tree。

Animate：标记动画以改变控件的状态（Tick animations to changes widget state）。
Build：为适应状态的改变，重建控件（Rebuild widgets to account for state changes）。
Layout：更新渲染对象的大小和位置（Update size and position for render objects）。
Paint：为已经合成完毕的layers记录显示列表（Record display lists for composited layers）。

## Widget & Element & RenderObject
类型 | Widget | Element | RenderObject | 说明 |
:---:|---|---|:---:|---|
组合型 | StatelessWidget | StatelessElement |  NA | Composer角色，将其他的widget进行拼装，组成一个新的Widget |
组合型 | StatefulWidget/State | StatefulElement |  NA | 同上 |
代理型 | ProxyWidget | ProxyElement |  NA | 数据传递 |
展示型 | RenderObjectWidget | RenderObjectElement |  RenderObject | 具有实际展示内容的视图 |
展示型 | SingleChildRenderObjectWidget | SingleChildRenderObjectElement |  同上 | 同上 |
展示型 | LeafRenderObjectWidget | LeafRenderObjectElement |  同上 | 同上 |
展示型 | MultiChildRenderObjectWidget | MultiChildRenderObjectElement |  同上 | 同上 |


## 创建树
1. 创建widget树
2. 调用`runApp（rootWidget）`将rootWidget传递给rootElement，作为rootElement的子节点，生成Element树，由Element树生成Render树。

Widget：存放渲染内容、视图布局信息，widget的属性最好都是immutable。

Element：存放上下文，通过Element遍历视图树，Element同时持有Widget和RenderObject。

RenderObjet：根据Widget的布局属性进行layout，绘制Widget传入的内容。

## 更新树
#### 为什么widget的属性都是immutable？
Flutter界面开发是一种响应式编程，主张 Simple is fast。Flutter设计的初衷是数据变更的时候发送通知到对应的可变更节点（可能是一个StatefulWidget子节点，也可能是rootWidget），由上到下重新创建Widget树进行刷新，这种思路比较简单，不用关心数据变更会影响哪些节点。

#### widget重新创建，element树和renderObject树是否重新创建？
widget只是一个配置数据结构，创建是非常轻量的，Flutter团队对widget的创建于销毁做了优化，不用担心整个widget树重新创建会带来性能问题。renderObjct负责View的绘制，设计layout、paint的操作，是一个真正被渲染的View，整个View树的重新创建会有很大的开销，所以widget重新创建并不会引起renderObject树的重新创建。

#### 树的更新规则
1. 找到widget对应的element节点，设置element为`dirty`，触发`drawFrame`。`drawFrame`会调用element的`performRebuild()`进行树的重建。
2. widget.build() == null，deactivate element.child，删除子树，**流程结束**。
3. element.child.widget == null，mount 新子树，**流程结束**。
4. element.child.widget == widget.build()，无需重建，否则进入流程5
5. widget.canUpdate(element.child.widget，newWidget) == true，更新child的slot，element.child.update（newWidget）（如果child还有子节点，则递归上面的流程进行子树创建），**流程结束**，否则进入流程6.
6. widget.canUpdate(element.child.widget，newWidget) == false（widget的classtype 或者 key不相等），deactivate element.child，mount 新子树

##### 注意事项
1. element.child.widget == widget.build()，不会触发子树的update，当触发update时，要注意widget是否使用旧的widget，没有new widget，导致update流程走到该widget就停止了。
2. 子树的深度变化，会引起子树重建，如果子树是一棵复杂度很高的树，可以用GlobalKey作为子树widget的key。GlobalKey具有缓存功能。


#### 如何触发树的更新
1. 全局更新：调用`runApp（rootWidget）`，Flutter启动时调用后，一般不会再调用。
2. 局部子树更新，让该子树继承StatefulWidget，并创建对应的State类实例，通过调用`state.setState()`来触发该子树更新。

## Widget
1. StatelessWidget：无中间状态变化，只根据初始内容进行创建的Widget。
2. StatefulWidget：存在中间状态变化，会根据内容变动更新的Widget。由于Widget都是immutable的，所以Flutter引入了State类用于存放中间状态，通过`setState()`来更新子树。

## State 生命周期
1. initState()：state 创建后，被加入到树中时调用。
2. didUpdateWidget(new Widget)：祖先节点rebuild widget时调用。
3. deactivate()：widget被移除的时候调用。一个widget从树中被移除，可以再dispose接口被调用之前，重新加入到一颗新树中。
4. didChangeDependencies()：
	- 初始化时，在initState()被调用后立刻调用；
	- 当依赖的InheritedWidget rebuild，会触发此接口被调用。
5. build()：
	- 调用`initState`后。
	- 调用`didUpdateWidget`后。
	- 当接收到`setState`调用后。
	- 当依赖这个状态的对象发生改变之后。（比如，一个InheritedWidget，被前置build引用，并发生了变化。）
6. dispose()：widget被彻底销毁时。
7. reassemble()：Hot Reload时。

##### 注意事项：
1. A页面Push到新的B页面，A页面的widget树中所有的state会依次调用deactivate()，disUpdateWidget(newWidget)，build()。
2. 当ListView中item滚动出可显示区域时，item会从树中remove，此item子树中的所有state都会被dispose()，state记录的数据会被销毁，当item滚回可显示区域时，会重新创建新的state、element、renderObject。
3. 使用Hot Reload功能时，要主要state实例有没有被重新创建，如果该state中存在复杂资源时，更新需要重新加载才能生效，那么在reassemble()中处理，不然可能会出现不可名状的bug。

## 数据流转

#### 从上往下
数据从根往下传输数据，常规做法是一层层往下传，当深度变大时，数据传输会变得困难，Flutter提供InheritedWidget用于子节点向祖先节点获取数据

```
class FrogColor extends InheritedWidget {
   const FrogColor({
     Key key,
     @required this.color,
     @required Widget child,
   }) : assert(color != null),
        assert(child != null),
        super(key: key, child: child);

   final Color color;

   static FrogColor of(BuildContext context) {
     return context.inheritFromWidgetOfExactType(FrogColor);
   }

   @override
   bool updateShouldNotify(FrogColor old) => color != old.color;
}
```

child及其以下节点可以通过`FrogColor.of(context).color`来获取color数据。

BuildContext就是Element的一个接口类。

context.inheritFromWidgetOfExactType(FrogColod)其实是通过context（element）往上便利书，查找到第一个FrogColor的节点，获得这个节点的widget对象。

#### 从下往上

子节点状态变更，通过发送通知的方式上报。

- 定义通知类，继承自Notification
- 父节点使用NotificationListener进行监听。
- 子节点有数据变化通过`Notification(data).dispatch(context)`上报。

## Layout
#### Size 计算
parent传入约束条件，在`drawFrame`的layout截断，child根据自身的渲染内容返回Size。

##### 注意事项
在`build()`截断获取不到size，很多时候需要提前获得部分Widget Size进行布局。

解决方案：当Widget在对应renderObject的layout阶段之后，发送一个LayoutChangeNotification（参考 SizeChangedLayoutNotifier class，但是SizeChangedLayoutNotifier没有上报init layout size，可以参考这个实现封装一个Notifier。

#### Offset 计算
1. renderObject拿到计算好的size，加上布局属性（align、padding）等，计算child相对parent的offset。
2. offset存在每个child renderObject的BoxParentData中。
3. 当parent拥有mutil children时，BoxParentData还用来存children兄弟节点之间的遍历顺序。

#### Relayout boundary
renderObject在layout阶段做了Relayout boundary优化，当子树进行relayout时，满足下面三种情况的一种

1. parentUsesSize == false
2. sizedByParent == true
3. constraints.isTight

那么这个renderObject将被设置为Relayout boundary，也就是这个renderObject重新layout不会触发parent的layout，一般情况下，开发人员不需要关心relayout boundary，除非使用CustomMultiChildLayout

## Paint

#### Layer
iOS每个UIView都有一个layer，Flutter的renderObject不一定存在layer。

一般情况下一个renderObject子树都渲染在一个layer上。
1. 当一个renderObject的`alwaysNeedsCompositing == true`或者`isRepaintBoundary == true`，renderObject会有对应的compositing layer。
2. 子renderObject会对目标layer返回对应的offsetLayer，目标compositing layer 再根据offset合成一个渲染的纹理buffer。


#### Repaint Boundary
类似Relayout Boundary，Paint阶段也有Repaint Boundary。目的和layout一样，就是对应子树的paint不会导致外部的repaint，但是Repaint Bondary需要开发人员设置，使用RepaintBoundary Widget进行设置，ListView的渲染Item默认都使用了RepaintBoundary，显而易见ListView的Children之间是互相独立的。

Flutter建议复杂的image渲染使用RepaintBoundary，image的渲染需要I/O操作，然后解码，最后渲染，使用RepaintBoundary可以进行GPU缓存。但缓存不一定会成功，Engine会判断这个image是否足够复杂，毕竟GPU缓存是有限的，同时RepaintBounday还会对一些反复渲染的layer进行缓存处理（反复渲染3次以上）。















