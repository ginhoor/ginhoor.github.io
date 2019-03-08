## 优点
#### 便利性
即使直接使用SQLite，在业务层也不应当直接操作SQL语句。数据库操作最终都是被封装起来使用的。这样直接操作SQLite和使用CoreData区别不大。

#### 存储性能
CoreData也是使用SQLite格式作为磁盘存储格式，所以性能上区别也不大。

#### 查询性能
打开CoreData的Debug模式，可以看CoreData具体执行了多少的SQL语句，可以进行优化。

#### 数据验证
可以设置简单的数据验证，也支持自定义数据验证。

#### 关系维护
支持关系级联删除，禁止删除等。

#### 迁移性
CoreData提供了轻量级迁移方案和自定义迁移方案，可以免去用SQL语句迁移数据。

#### 内存优化
1. CoreData支持延迟写入，避免密集的SQL操作同时发生。
2. CoreData在Fetch对象时，对象的关联对象将为fault类型，即懒加载想关联对象，这样可以避免内存占用过大。

#### 多线程优化
支持Merge Policies，例如多线程中，对同一个数据修改。

可以让两个Context共享一个持久化存储缓存（需要加互斥锁）。

两个Context，一个负责UI数据的获取，一个Context负责耗时数据的获取。当发生数据更新里，通过通知就可以刷新数据源。

#### 支持KVC和KVO

#### 支持redo和undo

## 缺点

#### 不适合跨平台
CoreData是Cocoa的一部分，用C实现。

#### 多人开发
多人合作开发的时候，由于CoreData是使用XML文件格式保存的，所以做merge会比较头疼。

#### 大量更新删除的操作效率较低
每次都需要先取到内存中

#### 主键需要自己维护
CoreData通过objectID来表示数据唯一性

#### 内存占用会比较高
需要维护ManagedContext，跟踪对象变化

## 参考资料
[iOS 持久化存储之CoreData VS 直接SQlite](https://blog.csdn.net/Hello_Hwc/article/details/46848855)

