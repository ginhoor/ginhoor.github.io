[iOS CoreData详解（四）Faulting and Uniquing](https://blog.csdn.net/hello_hwc/article/details/46005541)

## Faulting和Uniquing

1. Faulting是一种CoreData降低内存使用的机制，是惰性加载的一种。
2. Uniquing是辅助Faulting的机制，保证在一个managed object context中只有一个managed object来表达一条记录。

## Faulting限制对象图的大小
一个Faulting在内存中就是一个对象的占位符，这个占位符代表的对象并没有完全取到内存中。

1. 一个Managed Object的fault就是相关类的对象
2. 一个Relationship的falut表示对应集合的实例。这样的占位符降低了内存的使用，也不需要把fault对象的相关对象再取到内存中。

fault对于使用者是透明的，在用户获取对应的fault对象的属性时，CoreData会从磁盘再获取对应数据，这个过程称为Firing Faults。

## Firing Faults的过程

1. CoreData先从持久化存储协调器的Cache中查找获取。
2. Cache中没有数据，则通过fetch从自盘获取，再加入到Cache中。


## Faulting的性能优化
一次fire一个fault的效率很低。
CoreData提供了两种解决方案

1. Batch Faulting 一次处理一批
2. Pre-fetching 预提取。

## 把初始化的对象转换成Fault

把已经初始化的对象转换为fault的优点

1. 降低内存
2. 保证对象的数据都是最新的（多线程下）

把一个对象转为fault的方法`refreshObject:mergeChanges:`，会将对应的持久化属性设置为nil，断开相关对象的强引用。

## Uniquing

例如，两个Person对应有两个Job（fault）。在进行了firing faulting后，Job被取出，如果两个Job相同，则会自动指向一个对象。这样就可以减少内存。