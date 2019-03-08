## 原文
[Stateful or Stateless widgets?](https://flutterdoc.com/stateful-or-stateless-widgets-42a132e529ed)

## 前言
在创建一个Flutter App的时候，会遇到两种类型的控件

1. 有状态控件（stateful widgets）
2. 无状态控件（stateless widgets）

## Stateless Widgets
在创建控件的时候，有一些控件不需要管理控件内部状态，这个时候就可以选择无状态控件。无状态控件除了用数据初始化以外的时刻，不需要改变它的状态。

在Flutter中，比较常见的有Text，Raised Button，Icon。

### Texture

以Text控件为例（[源码](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/text.dart#L199)），这个控件没有可以被改变的状态。


```dart
 const Text.rich(
   TextSpan(
     text: 'Hello', // default text style
     children: <TextSpan>[
       TextSpan(text: ' beautiful ', style: TextStyle(fontStyle: FontStyle.italic)),
       TextSpan(text: 'world', style: TextStyle(fontWeight: FontWeight.bold)),
     ],
   ),
 )
```
Text使用一个构造函数进行初始化，初始化时设置一些属性用于构建控件和控制显示内容。

父控件通过设置alignment、direction、text，来控制自己的显示，和管理子控件的显示。


### 什么样的控件是stateless的

那什么时候该选择使用stateless的控件呢？

例子一：
创建一个自定义的ProgressBar控件，使用的时候通过几个参数初始化就能展现给用户。这个控件不需要保留任何状态，它将被添加到父控件的控件树中，或者从父控件的控件树中移除，它的父控件通过管理自己的状态来控制ProgressBar的显示或隐藏。

例子二：
创建一个内容列表中的列表子控件，比如一个蛋糕列表，蛋糕就是这个控件。这个控件将被填入蛋糕的信息，用于展示蛋糕。但这个控件不会保留状态，它只是使用填充的数据，通过父控件的设置来展示给用户。

从这些例子中可以看出，stateless的控件是非动态的。它只需要将数据传入其中，也就是说它只能通过构造函数传入的数据来控制如何显示。

## Stateful widgets
Statefule widgets是Stateless widgets的对立面，它是动态的。它可以被随时的、动态的修改内容，不像Stateles widgets只能通过构造函数设置。改变的方式可能是用户的输入，异步的响应或者对另一个状态的变化做出反应。

在Flutter中，比较常见的有Image、Form、Checkbox等。

### Image
以Image为例（[源码](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/image.dart)），Image控件的源文件看起来略有不同，它是继承自StatefulWidget类，也可以通过构造函数传入数据来进行初始化，这些都和StatelessWidget相同。

不同的地方是，这个控件中多了一个`createState`方法（[源码](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/image.dart#L575)）

```
_ImageState createState() => new _ImageState();
```
这个覆写的方法用来为控件创建状态。不用深入Image如何运作的源码，就可以发现_ImageState是用来保存不同的属性。

```
  ImageStream _imageStream;
  ImageInfo _imageInfo;
  bool _isListeningToStream = false;
  bool _invertColors;
```
`ImageInfo `用来为控件加载实际图片的属性。

`_handleImageChanged`函数（[源码](https://github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/image.dart#L641)）中使用了State类的setState函数，来表示状态发生了变更。

```
setState(() { 
    _imageInfo = imageInfo; 
});
```
当发生了状态变更时，控件将根据新状态重新构建，也就是说控件将加载更新后的imageInfo图片数据。这就是Image的动态行为的表现方式，它会一直监听图片引用，当引用放生变化时，它的状态也会改变。因此Image控件管理着它自己的状态，不依赖于父控件。

#### 什么样的控件是Stateful widgets
例子一：
创建一个控件用来显示一个记录是否被用户添加了书签。这个可以点击的控件将保留一个`isBookmarked`的属性，当控件被点击的时候，它的状态将会被修改，在控件的`build`函数中将会为记录设置上已经“已经添加过书签”的标志。

例子二：
创建一个控件用来保存当前被选中的记录数量，当点击“+”按钮的时候将会增加被选中的记录数量。这个控件将保存一个数量的状态，每次点击“+”按钮时，新的数量都会记录到State中，并且在`build`函数中更新显示为新的数量。