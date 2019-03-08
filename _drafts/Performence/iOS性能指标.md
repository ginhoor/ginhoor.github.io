## 性能指标

1. 启动时间
2. 内存占用优化，内存警告次数
3. CPU使用率
4. 页面渲染时间（从`viewDidLoad:`到`viewDidAppear:`的时间），刷新帧率
5. 网络请求时间，流量消耗量
6. UI阻塞次数，主线程阻塞超过400ms次数
7. 耗电功率

## 检测途径/指标采集

1. Xcode Instrument
2. 自行打点看日志
3. 第三方SDK

### 自行检测

1. 自行通过AOP切面的方式，将打点代码加入到ViewController的什么周期函数中，例如`viewDidLoad:`和`viewDidAppear:`
2. 上报时机的确定。选择合适的实际，一次性上报累计的统计数据。

## 优化

### 启动时间
1. 在Xcode > Edit scheme > Arguments > Eviroment Variables中增加`DYLD_PRINT_STATISTICS`环境变量就获得每个阶段的耗时。
2. 通过Instrument Time Profiler 找到`-[UIApplication _reportAppLaunchFinished]`的最后一帧，也能计算出启动时间。

### 内存
使用Instrument Allocations 获得内存使用量。

手动获取内存使用量

```
#import <mach/mach.h>
#import <mach/task_info.h>

- (unsigned long)memoryUsage
{
    struct task_basic_info info;
    mach_msg_type_number_t size = sizeof(info);
    kern_return_t kr = task_info(mach_task_self(), TASK_BASIC_INFO, (task_info_t)&info, &size);
    if (kr != KERN_SUCCESS) {
        return -1;
    }
    //单位：10-KB，20-MB 
    unsigned long memorySize = info.resident_size >> 10;

    return memorySize;
}
```

### CPU使用率

影响CPU的主要是密集计算

1. 动画
2. 布局计算、Autolayout
3. 文本计算、文本渲染
4. 图片解码、绘制

常见的优化方式有缓存TableView的Cell高度。
缓存和算法优化可以解决大部分的问题。

使用Instrument Activity Monitor 获得CPU使用率。

手动获取CPU使用率


```
- (float)cpu_usage
{
    kern_return_t           kr = { 0 };
    task_info_data_t        tinfo = { 0 };
    mach_msg_type_number_t  task_info_count = TASK_INFO_MAX;

    kr = task_info( mach_task_self(), TASK_BASIC_INFO, (task_info_t)tinfo, &task_info_count );
    if ( KERN_SUCCESS != kr )
        return 0.0f;

    task_basic_info_t       basic_info = { 0 };
    thread_array_t          thread_list = { 0 };
    mach_msg_type_number_t  thread_count = { 0 };

    thread_info_data_t      thinfo = { 0 };
    thread_basic_info_t     basic_info_th = { 0 };

    basic_info = (task_basic_info_t)tinfo;

    // get threads in the task
    kr = task_threads( mach_task_self(), &thread_list, &thread_count );
    if ( KERN_SUCCESS != kr )
        return 0.0f;

    long    tot_sec = 0;
    long    tot_usec = 0;
    float   tot_cpu = 0;

    for ( int i = 0; i < thread_count; i++ )
    {
        mach_msg_type_number_t thread_info_count = THREAD_INFO_MAX;

        kr = thread_info( thread_list[i], THREAD_BASIC_INFO, (thread_info_t)thinfo, &thread_info_count );
        if ( KERN_SUCCESS != kr )
            return 0.0f;

        basic_info_th = (thread_basic_info_t)thinfo;
        if ( 0 == (basic_info_th->flags & TH_FLAGS_IDLE) )
        {
            tot_sec = tot_sec + basic_info_th->user_time.seconds + basic_info_th->system_time.seconds;
            tot_usec = tot_usec + basic_info_th->system_time.microseconds + basic_info_th->system_time.microseconds;
            tot_cpu = tot_cpu + basic_info_th->cpu_usage / (float)TH_USAGE_SCALE;
        }
    }

    kr = vm_deallocate( mach_task_self(), (vm_offset_t)thread_list, thread_count * sizeof(thread_t) );
    if ( KERN_SUCCESS != kr )
        return 0.0f;

    return tot_cpu * 100.; // CPU 占用百分比
}
```

### 刷新帧率
刷新帧率可以通过Instrument Core Animation获得。
最快为每秒60帧。


