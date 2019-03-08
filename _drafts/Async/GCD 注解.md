## Grand Central Dispatch 常用方法

```objective-c
// 主线程队列
dispatch_queue_t queue = dispatch_get_main_queue();

// 串行队列
dispatch_queue_t queue = dispatch_queue_create("identify", DISPATCH_QUEUE_SERIAL);

// 释放队列，从iOS 6.0以后，就由ARC管理了，不需要手动调用。
dispatch_release(queue);
    
// 同步执行
dispatch_sync(queue, ^{
});

// 并发队列
dispatch_queue_t queue = dispatch_queue_create("identify", DISPATCH_QUEUE_CONCURRENT);

// 全局并发队列
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

// 异步执行任务
dispatch_async(queue, ^{
});

// 栅栏任务
dispatch_barrier_async(queue, ^{
    //只有执行完栅栏任务前的操作后，才执行栅栏任务，栅栏任务之后的任务在栅栏任务执行后执行，无需等待栅栏任务完成。
}
dispatch_barrier_sync(queue, ^{
    //只有执行完栅栏任务前的操作后，才执行栅栏任务，再执行栅栏任务后的任务
});

                       
// 延迟任务
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    // 两秒后会执行当前任务
});

// 一次性任务
static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
});

// 重复执行任务，10次
dispatch_apply(10, queue, ^(size_t index) {
});

// 创建任务组
dispatch_group_t group = dispatch_group_create();
// 任务组异步执行
dispatch_group_async(group, queue, ^{

});

// 当任务组任务都完成后回调
dispatch_group_notify(group, queue, ^{

});

// 等待任务组中任务全部完成，会阻塞线程
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);

// 手动管理任务，当任务数为0时，会触发dispatch_group_wait和dispatch_group_notify，需要成对使用
// 任务组任务数加一
dispatch_group_enter(group);
// 任务组任务数减一
dispatch_group_leave(group);

// 创建一个信号量，设置初始信号量
// 可以让一个异步任务变成同步任务
dispatch_semaphore_t signal = dispatch_semaphore_create(0);
// 发送信号量，信号量加一
dispatch_semaphore_signal(signal);
// 等待信号，信号量减一，当信号量为零是会一直等待，阻塞线程。
dispatch_semaphore_wait(signal, DISPATCH_TIME_FOREVER);
// 如果信号值等于0那么就和NSCondition一样，阻塞当前线程进入等待状态，如果等待时间未超过timeout并且dispatch_semaphore_signal释放了了一个信号值，那么就会消耗掉一个信号值并且向下执行。如果期间一直不能获得信号量并且超过超时时间，那么就会自动执行后续语句。

```
## 关于Dispatch Source
参考[Dispatch Source学习](https://www.jianshu.com/p/aeae7b73aee2)
Dispatch Source可用来监听以下六类事件：

Timer Dispatch Source：定时调度源。
Signal Dispatch Source：监听UNIX信号调度源，比如监听代表挂起指令的SIGSTOP信号。
Descriptor Dispatch Source：监听文件相关操作和Socket相关操作的调度源。
Process Dispatch Source：监听进程相关状态的调度源。
Mach port Dispatch Source：监听Mach相关事件的调度源。
Custom Dispatch Source：监听自定义事件的调度源。

Dispatch Source根据同类事件的不同操作细分为11个类型：

DISPATCH_SOURCE_TYPE_DATA_ADD
属于自定义事件，可以通过dispatch_source_get_data函数获取事件变量数据，在我们自定义的方法中可以调用dispatch_source_merge_data函数向Dispatch Source设置数据

DISPATCH_SOURCE_TYPE_DATA_OR：
属于自定义事件，用法同上面的类型一样。

DISPATCH_SOURCE_TYPE_MACH_SEND：
Mach端口发送事件。
DISPATCH_SOURCE_TYPE_MACH_RECV：
Mach端口接收事件。
DISPATCH_SOURCE_TYPE_PROC：
与进程相关的事件。
DISPATCH_SOURCE_TYPE_READ：
读取文件事件。
DISPATCH_SOURCE_TYPE_WRITE：
写入文件事件。
DISPATCH_SOURCE_TYPE_VNODE：
文件属性更改事件。
DISPATCH_SOURCE_TYPE_SIGNAL：
接收信号事件。
DISPATCH_SOURCE_TYPE_TIMER：
定时器事件。
DISPATCH_SOURCE_TYPE_MEMORYPRESSURE：
内存压力事件。

```objective-c
+ (dispatch_source_t)timer
{
    dispatch_source_t timerSource = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0));
    
    if (timerSource) {
        dispatch_time_t startTime = dispatch_time(DISPATCH_TIME_NOW, 0*NSEC_PER_SEC);
        NSString *desc = timerSource.description;
        // 源，开始时间，触发间隔，精度（默认纳秒，为0时系统提供最大精度）
        dispatch_source_set_timer(timerSource, startTime, NSEC_PER_SEC*1, 0);
        dispatch_source_set_event_handler(timerSource, ^{
            static NSInteger i = 0;
            i++;
            NSLog(@"timer-->%@,Task:%d",desc,i);
        });
        
        dispatch_source_set_cancel_handler(timerSource, ^{
            NSLog(@"定时器被取消");
        });
        
        dispatch_resume(timerSource);
    }
    return timerSource;
}

```





