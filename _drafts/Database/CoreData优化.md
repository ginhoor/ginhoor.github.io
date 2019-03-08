## 查询优化

查询情况分为三种

1. 对象在当前Managed Context中（尚在内存中）。
2. 对象在持久化存储协调器中。
3. 对象需要从sqlite文件读取（I/O读取）。

当内存允许的时候，能同时取出的对象，不要分多次取出。

## 设置合理的Predicates
1. 限制更大的筛选条件应当放在优先位置，更好地提高筛选效率。查询字符串速度比较慢，所以（age > 10 ）AND （name LIKE 'JAKE'）比（name LIKE 'JAKE'） AND（age > 10）更合理。
2. 只fetch需要的数据。少量的数据，用Fetch Limits来获取。当有大量数据读取需求时，创建两个Fetch，第一个Fetch获取目前UI需要的数据，第二个Fetch获取剩余更多的数据。

```
NSInteger index = 0;
NSFetchRequest *request = [[NSFetchRequest alloc] init];
[request setFectchList:100];
[request setFetchOffset:100*index];
```
## 设置合理的fetchBatchSize
Fetch Request默认的fetchBatchSize为0，即一次读取目标表的所有数据。一次性加载很多的数据是一件很危险的事情。

通常的做法是设置一个是显示数据量两倍的fetchBatchSize。这样取到的数据数量还是所有数据的数据，但是实际数据获取与fetchBatchSize设置的数量相同的数量。当访问超过获取的范围时，CoreData会先释放旧的数据对象，再请求新的数据对象，这个过程是自动的，相当于Table的Cell重用机制。

## 使用countForFetchRequest查询
使用`countForFetchRequest`来查询数据数量，而不是使用executeFetchRequest获取数据后，再获取数据数量。

## 使用Faulting

Faulting是一种CoreData降低内存使用的机制，是惰性加载的一种。通过NSFetchRequest的`returnsObjectsAsFaults`可以设置。

#### Faulting限制对象图的大小
一个Faulting在内存中就是一个对象的占位符，这个占位符代表的对象并没有完全取到内存中。

1. 一个Managed Object的fault就是相关类的对象
2. 一个Relationship的falut表示对应集合的实例。这样的占位符降低了内存的使用，也不需要把fault对象的相关对象再取到内存中。

#### 降低内存使用
1. 对不需要的对象re-faulting，调用`refreshObject:mergeChanges:`，让已经初始化的对象，放弃他的关联属性，并释放内存。
2. 设置includesPropertyValues为NO，保证惰性加载。
3. 可以调用NSManagedObjectContext的reset方法来清空缓存。
4. 不需要undo的工程，可以将undo manager设置为nil

## 大数据量的对象

大数据量的对象例如音乐，视频等，使用SQLite多为Coredata的持久化存储。SQLite在扩展到100GB的数量级后，依然可以提供很好的性能支持。

1. CoreData通过URL的形式来关联数据。
2. 通过合理的关系设计，让大数据量的对象只有在使用时才产生加载。

## 多线程优化

1. persistentStoreCoordinator <--- mainContext <--- privateContext
这个模式下总共两个Context，一个是主线程中使用的mainContext，一个是子线程的privateContext，privateContext.parentContext = mainContext。

	每当privateContext执行save操作的时候，它都会将变动传递给mainContext，并执行存入数据库的操作。此时就会在主线程中发生一次I/O操作，会阻塞UI线程。

2. persistentStoreCoordinator <--- backgroundContext <--- mainContext <--- privateContext

	这个模式下一共三个Context，privateContext.parentContext = mainContext, mainContext.parentContext = backgroundContext。
	当privateContext发生save操作的时候，会传递到mainContext，再传递到backgroundContext，最终在backgroundContext中执行I/O操作，将数据存入数据库。

3. persistentStoreCoordinator <--- mainContext

	persistentStoreCoordinator <--- privateContext

	当privateContext发生save操作的时候，会直接进行I/O操作，再将数据变成merge到mainContext中，保证两个Context数据同步。但是这种模式会产生大量的mergeChanges。
	
## 参考资料
[CoreData 多线程下NSManagedObjectContext的使用](https://blog.csdn.net/willmomo/article/details/19759413)

[CoreData性能优化笔记](https://medium.com/shidanqing/coredata性能优化总结-1ea9ba5f0a02)
