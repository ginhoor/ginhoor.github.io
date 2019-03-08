
## @protocol NSLocking
```objective-c
- (void)lock;
    加锁
- (void)unlock;
    解锁
@end
```
## NSLock : NSObject \<NSLocking\>

```objective-c
@private
    void *_priv;
}

- (BOOL )tryLock;
    尝试加锁
- (BOOL)lockBeforeDate:(NSDate *)limit;
    在指定时间之前加锁
@property (nullable, copy) NSString *name API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
    锁名
@end

```

## NSConditionLock : NSObject \<NSLocking\>
条件锁
```objective-c

@private
    void *_priv;
}

- (instancetype)initWithCondition:(NSInteger)condition NS_DESIGNATED_INITIALIZER;
    通过条件初始化锁，比如数量，枚举
@property (readonly) NSInteger condition;
    条件
- (void)lockWhenCondition:(NSInteger)condition;
    通过条件加锁
- (BOOL)tryLock;
    尝试加锁
- (BOOL)tryLockWhenCondition:(NSInteger)condition;
    尝试通过条件加锁
- (void)unlockWithCondition:(NSInteger)condition;
    通过条件解锁
- (BOOL)lockBeforeDate:(NSDate *)limit;
    指定日期之前加锁
- (BOOL)lockWhenCondition:(NSInteger)condition beforeDate:(NSDate *)limit;
    指定日期之前条件加锁
@property (nullable, copy) NSString *name API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
    名称
@end
```

## @interface NSRecursiveLock : NSObject \<NSLocking\>
    该锁可以被同一个线程多次获取，而不会导致死锁。
    常用与递归调用

```objective-c


@private
    void *_priv;
}

- (BOOL)tryLock;
    尝试加锁
- (BOOL)lockBeforeDate:(NSDate *)limit;
    在指定日期之前加锁
@property (nullable, copy) NSString *name API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));

@end
```

## NSCondition : NSObject \<NSLocking\>
    该锁在给定线程中充当锁和检查点。

```objective-c

@private
    void *_priv;
}

- (void)wait;
    等待
- (BOOL)waitUntilDate:(NSDate *)limit;
    在指定日期之前等待
- (void)signal;
    发起单个信号，唤醒一个等待它的线程
- (void)broadcast;
    广播，发起信号，唤醒所有等待它的线程
@property (nullable, copy) NSString *name API_AVAILABLE(macos(10.5), ios(2.0), watchos(2.0), tvos(9.0));
    名称
@end

```