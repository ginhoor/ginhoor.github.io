
## NSOperation : NSObject 
```objective-c
- (void)start;
    任务开始
- (void)main;
    任务的主要实现
@property (readonly, getter=isCancelled) BOOL cancelled;
    是否被取消
- (void)cancel;
    取消任务
    
@property (readonly, getter=isExecuting) BOOL executing;
    是否正在执行
@property (readonly, getter=isFinished) BOOL finished;
    是否结束
@property (readonly, getter=isConcurrent) BOOL concurrent; // To be deprecated; use and override 'asynchronous' below
    是否并发
@property (readonly, getter=isAsynchronous) BOOL asynchronous API_AVAILABLE(macos(10.8), ios(7.0), watchos(2.0), tvos(9.0));
    是否并发
@property (readonly, getter=isReady) BOOL ready;
    是否准备好

- (void)addDependency:(NSOperation *)op;
    添加依赖
- (void)removeDependency:(NSOperation *)op;
    移除依赖
@property (readonly, copy) NSArray<NSOperation *> *dependencies;
    依赖的线程组
typedef NS_ENUM(NSInteger, NSOperationQueuePriority) {
	NSOperationQueuePriorityVeryLow = -8L,
	NSOperationQueuePriorityLow = -4L,
	NSOperationQueuePriorityNormal = 0,
	NSOperationQueuePriorityHigh = 4,
	NSOperationQueuePriorityVeryHigh = 8
};
    优先级

@property NSOperationQueuePriority queuePriority;
    在队列中的优先级
@property (nullable, copy) void (^completionBlock)(void) API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));
    完成回调
- (void)waitUntilFinished API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));
    阻塞当前线程直到当前线程结束
@property double threadPriority API_DEPRECATED("Not supported", macos(10.6,10.10), ios(4.0,8.0), watchos(2.0,2.0), tvos(9.0,9.0));

@property NSQualityOfService qualityOfService API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
    线程在系统中的相对重要性
    typedef NS_ENUM(NSInteger, NSQualityOfService) {
        NSQualityOfServiceUserInteractive = 0x21,
            用于直接参与UI交互
        NSQualityOfServiceUserInitiated = 0x19,
            用于用户明确请求的工作，需要立即获得结果
        NSQualityOfServiceUtility = 0x11,
            用于执行用户不太需要立即获得结果的工作
        NSQualityOfServiceBackground = 0x09,
            用于非用户发起的，比如缓存，预读等工作。
        NSQualityOfServiceDefault = -1
            没有明确的服务重要性信息。会使用NSQualityOfServiceUserInteractive和NSQualityOfServiceUserInitiated之间的重要性。
    } API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));

@property (nullable, copy) NSString *name API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
    线程名称

@end
```


## NSBlockOperation : NSOperation
 ```objective-c

+ (instancetype)blockOperationWithBlock:(void (^)(void))block;
    根据任务工作内容初始化
- (void)addExecutionBlock:(void (^)(void))block;
    添加工作内容
@property (readonly, copy) NSArray<void (^)(void)> *executionBlocks;
    工作内容回调集合
@end
 ```

## NSInvocationOperation : NSOperation 
Swift未实现

```objective-c
- (nullable instancetype)initWithTarget:(id)target selector:(SEL)sel object:(nullable id)arg;
    
- (instancetype)initWithInvocation:(NSInvocation *)inv NS_DESIGNATED_INITIALIZER;

@property (readonly, retain) NSInvocation *invocation;

@property (nullable, readonly, retain) id result;

@end
```



## NSOperationQueue : NSObject

```objective-c
static const NSInteger NSOperationQueueDefaultMaxConcurrentOperationCount = -1;

- (void)addOperation:(NSOperation *)op;
    添加线程
- (void)addOperations:(NSArray<NSOperation *> *)ops waitUntilFinished:(BOOL)wait API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));
    添加线程，是否阻塞当前线程，需要等待。

- (void)addOperationWithBlock:(void (^)(void))block API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));
    用一个没有参数没有返回值的block，生成Operation，并添加到队列中

@property (readonly, copy) NSArray<__kindof NSOperation *> *operations;
    当前线程中所有的操作

@property (readonly) NSUInteger operationCount API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));
    当前线程中操作数量
@property NSInteger maxConcurrentOperationCount;
    最大并发数
@property (getter=isSuspended) BOOL suspended;
    是否挂起

@property (nullable, copy) NSString *name API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));
    队列名称

@property NSQualityOfService qualityOfService API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
    队列在系统中的重要性
@property (nullable, assign /* actually retain */) dispatch_queue_t underlyingQueue API_AVAILABLE(macos(10.10), ios(8.0), watchos(2.0), tvos(9.0));
    用于执行操作的GCD调度队列
- (void)cancelAllOperations;
    取消所有操作
- (void)waitUntilAllOperationsAreFinished;
    等待所有操作完成后结束队列
@property (class, readonly, strong, nullable) NSOperationQueue *currentQueue API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));
    当前队列
@property (class, readonly, strong) NSOperationQueue *mainQueue API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));
    主线程
@end

NS_ASSUME_NONNULL_END
```

