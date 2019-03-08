### Error connecting to the service protocol: HttpException: , uri = http://127.0.0.1:1024/ws

由于修改了初始的Xcode项目包名，包名不一致导致Flutter无法建立连接。

尚未找到可以解决的办法，目前的思路是初始化包名用测试包的包名，正式包另外建立Target来定义。


### type 'List<dynamic>' is not a subtype of type 'List<Widget>'
```
  List<Widget> _getEnterItem(var entrances) {
    var widgets = [];  //<----BUG
    for (var entrance in entrances) {
      widgets.add(Text(entrance));
    }
    return widgets;
  }
```
这段代码中，将动态类型的List传递给了只包含Widget的List，造成了数据类型不一致，即使内部元素类型一致也不行。

```
  List<Widget> _getEnterItem(var entrances) {
    List<Widget> widgets = [];   //为List指定了元素类型
    for (var entrance in entrances) {
      widgets.add(Text(entrance));
    }
    return widgets;
  }
```


### 保存了代码，但是数据没有刷新

`Flutter Hot Reload`时，初始函数并不会重新执行，如`void initState()`、
`main()`

这是需要使用`Flutter Hot Restart`，就可以刷新项目。


### No Material widget found. ListTile widgets require a Material widget ancestor.
使用Navigator.push跳转的页面时，页面的根控件必须使用Scaffold，不然就会报错。

Although you can create a navigator directly, it's most common to use the navigator created by a WidgetsApp or a MaterialApp widget. You can refer to that navigator with Navigator.of.

[Navigator-class](https://docs.flutter.io/flutter/widgets/Navigator-class.html)