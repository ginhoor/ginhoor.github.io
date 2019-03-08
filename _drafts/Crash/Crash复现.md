[如何定位Obj-C野指针随机Crash(一)：先提高野指针Crash率]
(https://cloud.tencent.com/developer/article/1070505)

[如何定位Obj-C野指针随机Crash(二)：让非必现Crash变成必现]
(https://cloud.tencent.com/developer/article/1070512)

[如何定位Obj-C野指针随机Crash(三)：如何让Crash自报家门]
(https://cloud.tencent.com/developer/article/1070528)

## Crash出现的原因

多数是由于多线程和野指针引起，有很大的随机性，需要日志帮助重现。

1. 对象释放后，内存没有改动，原有内存保存完好，可能未Crash（随机Crash）
2. 对象释放后，内存没有改动，但是成员对象已经被删除，可能Crash，可能Crash在访问成员对象的时候，或者出现逻辑错误（随机Crash）。
3. 对象释放后，内存发生改动，写入了不可访问数据，直接Crash（必现Crash）
4. 对象释放后，内存发生改动，写入了可以访问数据，可能不Crash，出现逻辑错误，简介访问到不可访问数据（随机Crash）。
5. 对象释放后，内存发生改动，写入了可以访问数据，但是再次执行访问的时候执行代码把别的数据破坏了（随机Crash，难度大，概率低）。
6. 对象释放后再次release（必现Crash）。


**开发阶段**

1. Xcode scheme > diagnostics > Enable Zombie Objects，可以利用Xcode的Zombie机制来排查，当发生野指针访问时，能够提示野指针的类。
2. Xcode scheme > diagnostics > Enable Scribble，可以让对象释放后的内存上填上不可访问数据，直接发生Crash。

当App未连接Xcode调试，则无法使用。

**测试阶段**

通过为已经释放的内存地址写入不可访问数据，手动引起Crash。
。

1. 覆写NSObject的dealloc、Runtime的object_dispose或者C的free方法。推荐覆写C库的free方法，这样可以覆盖一部分C的野指针问题。
2. 当free的时候为内存设置上不可访问数据，数据长度通过malloc_size(p)来确定，数据与Xcode相同为“0x55”（如果free，内存在被写入不可访问数据后，依然可能被其他内存数据覆盖，最终又变成了随机Crash。为了避免系统再次将数据写入这段内存，这里将不释放内存）。
3. 如果当前指针是一个class，创建一个野指针管理类，将对象指针的isa替换成管理类的isa，将管理类中记录原来对象的class类型，在管理类中实现消息转发，来捕获引起crash的class和selector。
4. 当被管理的数据内存大于阈值时，就释放一部分，当系统内存警告时，也要释放一部分。
