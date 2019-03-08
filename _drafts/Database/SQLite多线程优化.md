
[【Dev Club 分享】微信 iOS SQLite 源码优化实践](https://cloud.tencent.com/developer/article/1071451)

## 1.简单的SQLite优化
1. 修改SQLite参数，如page_size、cache_size等。
2. 改变业务层调用，如主线程操作dispatch到子线程。

## 2.多线程并发优化

#### SQLite多句柄方案
SQLite支持多线程（几乎）无锁的并发方案。

1. 开启配置`PRAGMA SQLITE_THREADSAFE=2`。
2. 确保同一个句柄同一时间只有一个线程在操作。
3. 开启WAL模式`PRAGMA journal_mode=WAL`（可选）。

此时写操作会先对驾到wal文件的末尾，而不是直接覆盖旧数据数据。当读操作开始时，会记下当前WAL的文件状态，只访问之前的数据。这就保证了多线程读与读、读与写之间可以并发进行。

#### Busy Retry方案
有了多句柄方案后，写与写之间仍然会相互阻塞。SQLite提供了Busy Retry方案，当阻塞发生的时候，会触发Busy Handler，此时让线程休眠一段时间后，重新尝试操作。尝试次数达到阈值时，返回`SQLITE_BUSY`错误码。

虽然Busy Retry方案基本解决了问题，但是对性能的利用还不够。在Retry过程中，休眠时间的长短和重试次数，是决定性能和操作成功率的关键。

1. 如果休眠次数太短或者重试次数太多，会空耗CPU资源。
2. 如果休眠时间过长，会造成等待时间太长。
3. 如果重试次数太少，则会降低操作成功率。

简单的方法就是修改休眠时间，尽最大可能缩短空耗的时间，但实际情况此方案并不成功。

#### SQLite中控制并发相关的原理
SQLite可以分为两部分

1. Core层。包括接口层，编译器，虚拟机。通过传入SQL语句，由编译器编译SQL生成虚拟机的操作码opcode。虚拟机基于生成的操作码，控制Backend后端行为（数据读写，索引建立，分配页空间等）。
2. Backend层。由B-Tree、Pager、OS三部分组成，实现数据库的读写主要逻辑。

OS层是对不同操作系统的系统调用抽象层，它实现了一个Virtual File System，将OS层的接口在编译时映射到对应的操作系统的系统调用上。锁的实现也在这里。

SQLite通过两个锁控制并发。第一个锁对应DB文件，通过5中状态管理；第二个所对应WAL文件，通过修改16-bit的unsigned short int的每一个bit进行管理。尽管锁的逻辑有一些复杂，但此处并不需关心。两种所最终都落在OS层的sqlite3OsLock、sqliteOsUnlock和sqlite3OsShmLock上具体实现。

他们在锁的实现上比较类似。比如lock操作在iOS上的实现：

1. 通过pthread_mutex_lock进行线程锁，防止其他线程介入。然后比较状态量，若当前状态不可跳转，则返回SQLITE_BUSY。
2. 通过fcntl进行文件锁，防止其他进程介入。若锁失败，则返回SQLITE_BUSY。

**SQLite的Busy Retry方案的原因也正是因此。文件锁没有线程锁类似pthread_cond_signal的通知机制。当一个进程的数据库操作结束时，无法通过锁第一时间通知其他线程进行尝试。因此只能通过多次休眠来尝试。**

#### 微信方案
iOS App是单进程的，没有多进程并发的需求。在iOS的特定场景下，可以舍弃兼容性，提高并发性。

新的方案，当OS层进行lock操作时：

1. 通过pthread_mutex_lock进行线程锁，防止其他线程介入。然后比较状态量，若当前状态不可跳转，则将当前期望跳转状态插入到一个FIFO的Queue尾部。最后通过线程进行pthread_cond_wait进入休眠状态，等待其他线程唤醒。
2. 忽略文件锁。

当OS层的unlock操作结束后：

1. 取出Queue头部的状态量，并比较当前状态能否跳转。如果能跳转，则通过pthread_cond_signal_thread_np唤醒对应线程尝试。

此方案可以在DB空闲的第一时间，通知其他正在等待的线程，降低CPU空耗。此外，由于Queue存在，当主线程被其他线程阻塞时，可以将主线程插入到Queue的头部，当其他线程发起唤醒通知时，主线程可以拥有更高的优先级，从而降低用户可感知的卡顿。

该方案上线后，卡顿检测系统检测到

1. 等待线程锁的造成的卡顿下降超过90%
2. SQLITE_BUSY的发生次数下降超过95%

## 3.I/O性能优化
#### mmap
mmap可以减少数据从kernel层到user层的数据拷贝，从而提高效率。

只需配置`PRAGMA mmap_size=XXX`即可开启mmap

SQLite不仅支持mmap，而且推荐使用。开启mmap后，SQLite性能将有所提升，但他只会对DB文件进行mmap，而WAL文件享受不到这个优化。

开启WAL模式后，写入数据会先被追加到WAL文件的尾部。待文件增长到一定长度后，SQLite会进行checkpoint。这个长度默认为1000个page_size，在iOS上约为3.9MB。而在多句柄下，对WAL的操作时并行的。如果某个句柄将WAL文件缩短了，没有一个通知机制能让其他句柄进行更新mmap内容。此时其他句柄使用mmap操作被缩短后的内容，会造成Crash。而普通的I/O接口，只会返回错误。因此SQLite没有实现对WAL的mmap。

另外，文件从新增长，对于文件系统来说，需要消耗时间重新寻找合适的文件块。

#### 微信方案
1. 数据库关闭并checkpoint成功是，不在truncate或删除WAL文件，值修改WAL文件头的Magic Number。下次数据库打开时，SQLite会识别到WAL文件不可用，重新从头开始写入。
2. 为WAL添加mmap的支持。它在这个场景下是不会缩短的，那么不能mmap的条件就被打破了。实现上，只需在WAL文件打开时，用unixMapfile将其映射到内存中，SQLite的OS层即会自动识别，将普通的I/O接口切换到mmap上。

## 4. 其他优化
#### 禁用文件锁
iOS App并没有多进程的需求，因此可以直接注释掉os_unix.c中所有文件锁相关的操作

SQLite中的Cache机制。被加载到内存的page，使用完毕后不会立即释放。而是在一定范围内通过LRU算法更新page cache。也就是说只要Cache设置合适，大部分操作不需要读取新的page。因为文件锁的存在，本来只需要内存读取的，不得不进行I/O读取。

#### 禁用内存统计锁


SQLite会对申请的内存进行统计，而统计数据是放在同一个全局变量中进行计算的，这就需要加线程锁。

内存申请虽然不是很耗时，但是很频繁。如果不需要内存统计的特性，可以通过`sqlite3_config(SQLITE_CONFIG_MEMSTATUS, 0)`进行关闭