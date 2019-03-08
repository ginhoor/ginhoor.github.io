## NSOperationQueue的工作流程

1. NSOperationQueue在初始化是，可以指定并发数
2. 有NSOperation被加入到队列中，队列开始排期执行。
3. 当NSOperation被执行完毕，会用列表中被移除。
4. 当所有的NSOperation都被执行完，队列任务完成。

## 自定义NSOperation并发

NSOperation有三种状态：Ready、Executing、Finished。
需要覆写NSOperation的Executing、Finished属性，实现start方法，main方法、isConCurrent方法，isConcurrent方法需要返回YES。

1. 将Operation加入队列等待，此时为Ready状态
2. Operation轮到执行，在start方法中先检测Operation是否被取消，如果被取消则将isFinished设置为YES，结束Operation并从队列中移除。
3. Operation在start方法中执行准备工作，用NSThread创建线程调用main方法，并将状态设置为isExecuting。
4. Operation调用main方法开始工作，执行过程按需要加锁。
5. Operation工作结束后，将isExecuting设置为NO，将isFinished设置为YES，并从队列中移除。
## Grand Central Dispatch

NSOperation作为GCD的高级分装，即GCD可以实现NSOPeration所有的功能。
1. **串行队列**和**并行队列**分别组合**同步任务**和**异步任务**，
    dispatch_queue_t，dispatch_sync，dispatch_async
2. 延迟任务，dispatch_after
3. 一次性任务，dispatch_once
4. 重复执行任务，dispatch_apply
5. 按任务组分配任务，dispatch_group_t，dispatch_group_async，dispatch_group_sync
   1.  任务组完成后回调，dispatch_group_notify
   2.  任务组阻塞线程，直至完成。dispatch_group_wait
6. 信号量控制，dispatch_semaphore_t，dispatch_semaphore_signal，dispatch_semaphore_wait 
7. 事件监听，Dispatch Source
  1.Timer Dispatch Source：定时调度源。
  2.Signal Dispatch Source：监听UNIX信号调度源，比如监听代表挂起指令的SIGSTOP信号。
  3.Descriptor Dispatch Source：监听文件相关操作和Socket相关操作的调度源。
  4.Process Dispatch Source：监听进程相关状态的调度源。
  5.Mach port Dispatch Source：监听Mach相关事件的调度源。
  6.Custom Dispatch Source：监听自定义事件的调度源。

## Dispatch Source实现NSTimer

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



## 自旋锁
  当使用了自旋锁，获得锁的线程会一直执行，直到显示释放锁，其他线程会一直循环等待自旋锁释放。
  自旋锁容易引起优先级翻转，当低优先级线程获得锁之后，高优先级的线程只能进入忙等状态，直到锁被释放。