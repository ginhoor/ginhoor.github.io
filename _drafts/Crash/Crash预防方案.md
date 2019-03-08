[大白健康系统--iOS APP运行时Crash自动修复系统]
(https://neyoufan.github.io/2017/01/13/ios/BayMax_HTSafetyGuard/)


## 常见引起Crash的原因

1. Unrecognized Selector Crash
2. KVO Crash
3. 数组越界，插入nil，如用nil构造NSAttributeString
4. 访问野指针Crash
5. 非主线程操作UI
6. NSTimer Crash与内存泄露
7. NSNotification Crash

### Unrecognized Selector Crash

##### 原理
当对一个对象发送一个调用不存在方法的消息时，会引起崩溃。
原理：

	1. 当消息发起时，Runtime会先从调用对象注册的缓存函数hash表中查找对应的方法，如果存在，则调用实现。
	2. 如果没有查到，则会进入对象的方法列表中查询方法，如果存在，则调用实现。
	3. 如果没有查到，则会进入对象的父类中递归查找方法。
	4. 如果还没有查到，则会转向拦截调用，走消息转发机制。
	5. 如果没有重写拦截调用方法，则发生Crash

此时我们的方向是通过重定向、动态创建方法、动态替换方法实现来预防crash。

```objc
// 处理成员方法
+ (BOOL)resolveClassMethod:(SEL)sel;
// 处理类方法
+ (BOOL)resolveInstanceMethod:(SEL)sel;
```

系统调用`resloveClassMethod:/resolveInstanceMethod:`来允许动态添加消息实现。

实现这个方法，需要创建不属于类的冗余方法。

```objc
重定向处理
- (id)forwardingTargetForSelector:(SEL)aSelector;
```

系统调用`forwardingTargetForSelector:`来重定向到别的对象执行函数。

实现这个方法，需要将方法转发给另一个对象。


```objc
- (void)forwardInvocation:(NSInvocation *)anInvocation;
```

系统调用`forwardInvocation:`来将目标函数以其他形式实现。
实现这个方法，需要通过NSInvocation的方式将消息转发给多个对象，需要创建新的NSInvocation对象。

系统的调用顺序为`resloveClassMethod:/resolveInstanceMethod:`>`forwardingTargetForSelector:`>`forwardInvocation:`

从开销上来看，选择`forwardingTargetForSelector:`来实现捕获比较合适。

**预防Crash方案**

1. 替换NSObject`forwardingTargetForSelector:`的实现，注入捕获Crash的函数
2. 因为`forwardInvocation:`的执行顺序晚于`forwardingTargetForSelector:`，所以先判断对象是否实现了`forwardInvocation:`，如果实现，则不处理。
3. 捕获函数中，将消息转入拦截类对象中进行处理，并返回这个对象作为响应对象。
4. 拦截类中需要实现`resolveClassMethod: `方法，当被访问时，动态的为类中创建方法。

### KVO Crash
**原理**

1. 被观察者在dealloc，但是KVO未释放。
2. 观察者dealloc，但是KVO未释放。
3. 重复添加或者移除观察者（尤其是多线程操作）。

**预防Crash方案**

创建一个delegate来管理KVO，运用method swizzling来注入方法。

1. `addObserver:forKeyPath:options:context:`创建KVO的时候，如果没有重复添加，则记录观察者和观察的属性（使用NSHashTable弱引用记录观察者）。
2. `removeObserver:forKeyPath:`移除KVO的时候，查找delegate中存储的观察关系，如果不存在则取消操作，防止重复移除。
3. `observeValueForKeyPath:ofObject:change:context:`取值的时候，先从观察关系中查找。如果查到观察者，则调用观察者的对应方法，如果不存在观察者，则需移除这段KVO关系。
4. `dealloc`释放对象时，检测该对象的KVO关系，如果存在被观察的KVO，则移除这些KVO关系。

### NSNotification Crash
**原理**

当为一个对象注册了Notification接收，但在dealloc的时候未注销，会引起Crash。

不过iOS9之后Apple已经做了优化，不在会引起Crash

**预防Crash方案**

1. 在NSNotificationCenter使用`addObserver:selector:name:object:`注册观察者时，为observer添加标记。
2. 在观察者`dealloc`释放对象时，判断observer标记，如果被注册为Notification Oberserver，则调用`[[NSNotificationCenter defaultCenter] removeObserver:self]`注销自己。


### NSTimer Crash与内存泄露
**原理**

当使用`scheduledTimerWithTimeInterval:target:selector:userInfo:repeats:`创建NSTimer的时候，NSTimer会强引用target实例，所以当NSTimer未释放的时候，即可能引起target无法释放，导致内存泄露。

如果NSTimer是最后一个强保持target的实例，当NSTimer被`invalidate`后，target将会被释放，如果在`invalidate`之后再访问target，会造成野指针Crash。

**预防内存泄露方案**

为NSTimer的target增加一个delegate来控制弱引用实际target。

1. 在使用`scheduledTimerWithTimeInterval:target:selector:userInfo:repeats:`时，使用delegate来包装target。
2. 当NSTimer调用target方法时，实际调用的是delegate，所以需要将delegate的方法调用转向target。
3. 当NSTimer被释放的时候，delegate也会随之释放，target也不会被NSTimer强引用。


### 数组越界，插入nil

**原理**

例如NSArray、NSMutableArray、NSDictionary、NSMutableDictionary是不允许插入nil值；

NSArray、NSMutableArray不允许越界访问；NSMutableString不允许用nil构造等。


**预防Crash方案**

用Method swizzling替换原有方法实现，比如在NSMutableArray的`addObject:`方法中加入对nil的判断。

注意例如NSArray是一个抽象类，我们需要通过`objc_getClassList`、`class_getSuperclass`来获得所有的直接子类，并替换他们的方法。

### 访问野指针Crash

**原理**

一个指针指向对象，当这个对象释放掉的时候，指针未被设置为nil，而是仍然指向对象的内存地址。当再次访问这个指针的时候，就会发生Crash。

测试阶段可以使用Xcode Zombie Objects来来排查，当发生野指针访问时，能够提示野指针的类。但是当App未连接Xcode调试，则无法使用。

**预防Crash方案**

实现一个类似Zombie的机制，加上对Zombie实例的所有方法拦截机制和消息转发机制，这样当访问野指针Crash的时候就不会Crash，并能记录Crash。

为了记录Zombie并且不Crash，则需要保持Zombie对象（延迟释放），会导致内存消耗，需要合理管理Zombie对象的内存空间，以及制作清理方案。

此方案能一定程度上解决野指针的问题，但是由于内存空间有限，当Zombie对象由于内存不足被释放后，防护功能也就失效了。


1. 修改NSObject的`allocWithZone:`方法，加入判断是否需要野指针防护，如果需要保护，则为类加上flag标识。
2. 修改NSObject的`dealloc`方法，当对象进入释放阶段并且开启了野指针防护，则将类并作为关联对象绑定到类上，然后将对象的isa指针指向Zombie对象，再调用`objc_destructInstance `方法来释放对象的相关实例。
3. 当Zombie对象存储容量达到上限时，需要先将旧对象进行释放，再插入新的Zombie对象。


### 非主线程操作UI
开启 Xcode scheme > diagnostics > Main Thread Checker，当发生非主线程的UI操作时，Xcode就能捕获到。
如果要预防，可以重写UIView、CALayer的方法如`setNeedsLayout`，`setNeedsDisplay`，`setNeedsDisplayInRect`





