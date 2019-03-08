##### SDWebImage与AFNetworking的区别
AFNetworking偏重于HTTP请求构建，请求结果解析等数据处理。SDWebImage偏重于下载队列，图片缓存等缓存管理。

#####   调用UIImage+WebCache的图片设置方法方法

1. 如果禁止缓存，则取消当前缓存操作。
2. 没有设置延迟显示占位图，则在主线程设置占位图显示
3. URL未设置，则返回错误信息。
4. 从context中获得SDWebImageManager，如果不存在，则使用全局Manager
5. 创建SDWebImageOperation
6. 当下载完成
   1. 没有下载到图片，直接执行完成回调
   2. 下载到了图片，在完成回调中返回图片数据

##### 创建SDWebImageOperation 图片获取流程

1. 创建一个SDWebImageCombinedOperation
2. 将manager关联到operation
3. 检测是否是已经失败过的URL并且未开启失败重试，或URL长度是否为0，如果成立，则返回错误，结束流程
4. 将operation加入队列中。
5. 根据URL获得CacheKey
6. 创建cacheOperation的缓存操作(cacheOperation)，根据key在缓存中查询缓存结果
7. 当cacheOperation查询操作完成时
    1. 如果线程已经被取消，将cacheOperation从队列中移除，并结束
    2. 检测是否需要下载，下载后是否需要刷新缓存，如果成立，则执行完成回调，将当前缓存数据传出。
        3.如果需要下载，创建downloadToken的下载操作(downloadToken)
        1. 当下载完成后
        2. 如果线程被取消，则结束流程。
        3. 如果出现错误，则执行**完成回调**并抛出错误
            1. 如果实现了协议中的shouldBlockFailedURL方法，则会调用对应协议方法，如果协议方法回调为YES，则将URL放入失败URL缓存。
            2. 如果未实现协议方法，则根据错误是否为无法连接到网络，超时，未授权等判断将URL放入失败URL缓存。 
        4. 如果允许失败重试下载，则将URL从失败URL缓存中移除。
        5. 图片下载成功并且需要转换
            1. 判断图片要自定义转换，如果有则执行自定义图片转换
            2. 将图片，缓存数据，key存入缓存，如果有自定义缓存构造器，则执行
            3. 执行**完成回调**
        5. 图片下载成功，如果有自定构造器，则执行构造器后的数据后存入缓存，如果没有，则直接存入缓存。
        6. 执行**完成回调**
        7. 如果完成，则将operation从队列中移除
    3. 存在缓存图片
        1. 执行**完成回调**，将operation移除队列
          1. 如果不存在缓存图片，也不需要下载，执行**完成回调**，返回空值
8. 在队列中的Task轮到时，开始执行。

### SDWebImageDownloader下载图片
1. 根据URL从缓存中查找operation
    1. 如果不存在，则创建operation，在下载完成回调中删除operation。
2. 将operation加入到当前下载缓存中。
3. 将operation加入到当前下载队列中
4. 构造downloadToken返回

## 相关资料

### SDWebImageManagerDelegate
```objective-c
- (BOOL)imageManager:(nonnull SDWebImageManager *)imageManager shouldDownloadImageForURL:(nullable NSURL *)imageURL;
    控制当缓存不存在的时候，是否应当下载图片

- (BOOL)imageManager:(nonnull SDWebImageManager *)imageManager shouldBlockFailedURL:(nonnull NSURL *)imageURL withError:(nonnull NSError *)error;
    控制当下载失败的时候，是否标记URL为已失败状态
    如果实现了此方法，则不会使用内置方式根据错误码来标记URL

- (nullable UIImage *)imageManager:(nonnull SDWebImageManager *)imageManager transformDownloadedImage:(nullable UIImage *)image withURL:(nullable NSURL *)imageURL;
    是否转换图片，在缓存到磁盘和内存前。
    此方法在全局队列使用，注意不要阻塞主线程
```
### SDWebImageOptions
```objective-c
typedef NS_OPTIONS(NSUInteger, SDWebImageOptions) {
    SDWebImageRetryFailed = 1 << 0,
        默认当URL下载完成，并且失败了，URL将进入黑名单不再调用。
        此选项将启动失败的URL再次加载。

    SDWebImageLowPriority = 1 << 1,
        默认图片下载是在UI交互过程中启用的。
        此选项将禁用这个特性，比如引起UIScrollView减速
    
    SDWebImageCacheMemoryOnly = 1 << 2,
        此选项将禁用磁盘缓存，只用内存缓存
    
    SDWebImageProgressiveDownload = 1 << 3,
        此选项将开启渐进式下载，下载过程中会逐步显示图片。
        默认图片只在下载完成后展示
    
    SDWebImageRefreshCached = 1 << 4,
        默认磁盘缓存是由NSURLCache完成。
        此选项将优化相同URL的请求加载。
        如果缓存图片刷新了，则完成回调会被调用两次，第一次是缓存图片，第二次是最终图片。
        只有当URL不是静态的时候才使用
    
    SDWebImageContinueInBackground = 1 << 5,
        开启图片的后台下载，如果后台任务超时，则会取消下载操作
    
    SDWebImageHandleCookies = 1 << 6,
        是否处理Cookie
        
    SDWebImageAllowInvalidSSLCertificates = 1 << 7,
        允许无效证书连接
        
    SDWebImageHighPriority = 1 << 8,
        图片加载将被优先处理
        
    SDWebImageDelayPlaceholder = 1 << 9,
        默认占位图会在图片下载过程中显示。
        此选项将使占位图在图片下载完成后显示，如图片下载失败。
        
    SDWebImageTransformAnimatedImage = 1 << 10,
        一般不经常调用transformDownloadImage delegate方法，在动画动态图上
        此选项将开启每次都调用
    
    SDWebImageAvoidAutoSetImage = 1 << 11,
        默认图片下载完成后就会被设置到ImageView
        开启此选项后，你需要在下载完成后手动设置图片
    
    SDWebImageScaleDownLargeImages = 1 << 12,
        默认图片会根据原始大小解压
        在iOS上，开启此选项，将根据设备来设置图片的缩放尺寸，比如@2x的图的scale为2
        如果设置了渐进式下载，则此选项不生效。
    SDWebImageQueryDataWhenInMemory = 1 << 13,
        默认内存中存在图片缓存时不会去查询磁盘缓存。
        开启此选项会强制同时查询磁盘缓存，开启此选项时推荐开启SDWebImageQueryDiskSync。
    
    SDWebImageQueryDiskSync = 1 << 14,
        默认查询内存缓存是同步的，查询磁盘缓存是异步的。
        此选项可强制磁盘缓存查询也为同步。
        当值禁用内存缓存时，此选项可避免TableCell在加载图片过程中闪烁。
    
    SDWebImageFromCacheOnly = 1 << 15,
    默认当缓存不存在的时候，图片会从网络进行下载。
    开启此选项可以防止网络加载，只用缓存。
    
    SDWebImageForceTransition = 1 << 16
        默认当图片加载完成后，使用SDWebImageTransition做一些视图转换的时，转换只能作用在网络下载的图片上。
        开启此选项会强制内存缓存和磁盘缓存也同样使用视图转换。
};
```
### SDWebImageManager：NSObject
```objective-c
@property (weak, nonatomic, nullable) id <SDWebImageManagerDelegate> delegate;
    代理

@property (strong, nonatomic, readonly, nullable) SDImageCache *imageCache;
    图片缓存

@property (strong, nonatomic, readonly, nullable) SDWebImageDownloader; *imageDownloader;
    控制图片下载

@property (nonatomic, copy, nullable) SDWebImageCacheKeyFilterBlock cacheKeyFilter;
    根据URL构造一个CacheKey，默认使用URL的完整地址

@property (nonatomic, copy, nullable) SDWebImageCacheSerializerBlock cacheSerializer;
    解码后的图片或下载的数据转换成存到磁盘的实际数据。
    如果为nil，则从图像实例生成数据。此方法全局队列调用，注意不要阻塞主线程

@property (strong, nonatomic, readwrite, nonnull) SDImageCache *imageCache;
    图片缓存

@property (strong, nonatomic, readwrite, nonnull) SDWebImageDownloader *imageDownloader;
    图片下载管理

@property (strong, nonatomic, nonnull) NSMutableSet<NSURL *> *failedURLs;
    失败的请求URL

@property (strong, nonatomic, nonnull) dispatch_semaphore_t failedURLsLock;
    failedURLs线程锁，保证线程安全

@property (strong, nonatomic, nonnull) NSMutableSet<SDWebImageCombinedOperation *> *runningOperations;
    下载线程队列

@property (strong, nonatomic, nonnull) dispatch_semaphore_t runningOperationsLock;
    线程队列锁

+ (nonnull instancetype)sharedManager;
    全局实例
- (nonnull instancetype)initWithCache:(nonnull SDImageCache *)cache downloader:(nonnull SDWebImageDownloader *)downloader NS_DESIGNATED_INITIALIZER;
    根据Cache和Downloader来创建一个实例

- (nullable id <SDWebImageOperation>)loadImageWithURL:(nullable NSURL *)url options:(SDWebImageOptions)options progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock completed:(nullable SDInternalCompletionBlock)completedBlock;
    图片下载
    如果不存在缓存，则根据URL进行图片下载，如果存在缓存则返回缓存版本。
    
- (void)saveImageToCache:(nullable UIImage *)image forURL:(nullable NSURL *)url;
    将图片加入缓存

- (void)cancelAll;
    取消所有当前队列
    
- (BOOL)isRunning;
    当前是否有队列在执行

- (void)cachedImageExistsForURL:(nullable NSURL *)url
                     completion:(nullable SDWebImageCheckCacheCompletionBlock)completionBlock;
    异步查询图片缓存

- (void)diskImageExistsForURL:(nullable NSURL *)url
                   completion:(nullable SDWebImageCheckCacheCompletionBlock)completionBlock;
    异步查询图片磁盘缓存
    
- (nullable NSString *)cacheKeyForURL:(nullable NSURL *)url;
    根据URL来获取CacheKey
```

### SDWebImageCombinedOperation
```objective-c

@property (assign, nonatomic, getter = isCancelled) BOOL cancelled;
    是否取消
@property (strong, nonatomic, nullable) SDWebImageDownloadToken *downloadToken;
    下载Token
@property (strong, nonatomic, nullable) NSOperation *cacheOperation;
    缓存线程
@property (weak, nonatomic, nullable) SDWebImageManager *manager;
    管理者
    
- (void)cancel
    取消缓存线程，取消下载线程，从队列中移除
```

### SDImageCacheConfig
```objective-c
@property (assign, nonatomic) BOOL shouldDecompressImages;
    是否解压图片，默认解压
    可以使图片渲染加速，但是会占用一定的内存，如果内存不足，请设置为NO

@property (assign, nonatomic) BOOL shouldDisableiCloud;
    是否禁用iCloud，默认为YES

@property (assign, nonatomic) BOOL shouldCacheImagesInMemory;
    是否使用内存缓存

@property (assign, nonatomic) BOOL shouldUseWeakMemoryCache;
    是否启用弱内存缓存方案。默认开启
    当内存不足警告触发时，即使内存缓存本身被清理了，一些由ImageView等强保持的实例可能再次恢复这些缓存，可以避免磁盘缓存重复查询，或者网络获取。
    可以避免当应用程序进入后台并被清理内存的时候，重新唤醒导致TableCell闪烁。
    此项可以动态修改

@property (assign, nonatomic) NSDataReadingOptions diskCacheReadingOptions;
    磁盘缓存读取选项

@property (assign, nonatomic) NSDataWritingOptions diskCacheWritingOptions;
    磁盘缓存写入选项

@property (assign, nonatomic) NSInteger maxCacheAge;
    最大缓存时间

@property (assign, nonatomic) NSUInteger maxCacheSize;
    最大缓存容量

/**
 * The attribute which the clear cache will be checked against when clearing the disk cache
 * Default is Modified Date
 */
@property (assign, nonatomic) SDImageCacheConfigExpireType diskCacheExpireType;
    磁盘缓存的到期时间选择类型，磁盘缓存清理时的清理时检查
    typedef NS_ENUM(NSUInteger, SDImageCacheConfigExpireType) {

    SDImageCacheConfigExpireTypeAccessDate,
        图片访问日期
    SDImageCacheConfigExpireTypeModificationDate
        图片修改日期
};

```
### SDImageCache
```objective-c
#pragma mark - Properties

@property (nonatomic, nonnull, readonly) SDImageCacheConfig *config;
    缓存配置

@property (assign, nonatomic) NSUInteger maxMemoryCost;
    图片缓存在内存中的占用量，以像素在内存中的数量决定

@property (assign, nonatomic) NSUInteger maxMemoryCountLimit;
    在内存缓存中的最大对象数

#pragma mark - Singleton and initialization

+ (nonnull instancetype)sharedImageCache;
    全局实例

- (nonnull instancetype)initWithNamespace:(nonnull NSString *)ns;
    用一个名字创建实例

- (nonnull instancetype)initWithNamespace:(nonnull NSString *)ns
                       diskCacheDirectory:(nonnull NSString *)directory NS_DESIGNATED_INITIALIZER;
    用名字和磁盘路径创建缓存

#pragma mark - Cache paths

- (nullable NSString *)makeDiskCachePath:(nonnull NSString*)fullNamespace;
    获取缓存存储路径

- (void)addReadOnlyCachePath:(nonnull NSString *)path;
    创建只读缓存地址，用于预载bundle图片。
#pragma mark - Store Ops

- (void)storeImage:(nullable UIImage *)image
            forKey:(nullable NSString *)key
        completion:(nullable SDWebImageNoParamsBlock)completionBlock;
    异步存储图片缓存进入内存缓存和磁盘缓存
    
- (void)storeImage:(nullable UIImage *)image
            forKey:(nullable NSString *)key
            toDisk:(BOOL)toDisk
        completion:(nullable SDWebImageNoParamsBlock)completionBlock;
    异步存储图片缓存
    
- (void)storeImage:(nullable UIImage *)image
         imageData:(nullable NSData *)imageData
            forKey:(nullable NSString *)key
            toDisk:(BOOL)toDisk
        completion:(nullable SDWebImageNoParamsBlock)completionBlock;
    异步存储图片缓存


- (void)storeImageDataToDisk:(nullable NSData *)imageData forKey:(nullable NSString *)key;
    同步存储缓存进入内存缓存和磁盘缓存

#pragma mark - Query and Retrieve Ops

- (void)diskImageExistsWithKey:(nullable NSString *)key completion:(nullable SDWebImageCheckCacheCompletionBlock)completionBlock;
    异步检测图片磁盘缓存是否存在

- (BOOL)diskImageDataExistsWithKey:(nullable NSString *)key;
    同步检测图磁盘缓存是否存在

- (nullable NSData *)diskImageDataForKey:(nullable NSString *)key;
    同步获取图片磁盘缓存

- (nullable NSOperation *)queryCacheOperationForKey:(nullable NSString *)key done:(nullable SDCacheQueryCompletedBlock)doneBlock;
    异步获取图片缓存

- (nullable NSOperation *)queryCacheOperationForKey:(nullable NSString *)key options:(SDImageCacheOptions)options done:(nullable SDCacheQueryCompletedBlock)doneBlock;
    异步获取图片缓存，可选缓存类型

- (nullable UIImage *)imageFromMemoryCacheForKey:(nullable NSString *)key;
    同步获取图片内存缓存
- (nullable UIImage *)imageFromDiskCacheForKey:(nullable NSString *)key;
    同步获取图片磁盘缓存

- (nullable UIImage *)imageFromCacheForKey:(nullable NSString *)key;
    同步获取图片缓存，先检测内存缓存，再检测磁盘缓存
#pragma mark - Remove Ops

- (void)removeImageForKey:(nullable NSString *)key withCompletion:(nullable SDWebImageNoParamsBlock)completion;
    移除缓存
- (void)removeImageForKey:(nullable NSString *)key fromDisk:(BOOL)fromDisk withCompletion:(nullable SDWebImageNoParamsBlock)completion;
    移除缓存

#pragma mark - Cache clean Ops

- (void)clearMemory;
    清理内存缓存
- (void)clearDiskOnCompletion:(nullable SDWebImageNoParamsBlock)completion;
    清理磁盘缓存

- - (void)deleteOldFilesWithCompletionBlock:(nullable SDWebImageNoParamsBlock)completionBlock;
    根据访问日期或者修改日期，删除超过最大保存天数的文件
    如果删除后还是超过了最大磁盘缓存容量，则从最老的文件开始删除，直至清理出一半的缓存空间。

#pragma mark - Cache Info

- (NSUInteger)getSize;
    获得磁盘缓存大小

- (NSUInteger)getDiskCount;
    获得磁盘缓存数量

- (void)calculateSizeWithCompletionBlock:(nullable SDWebImageCalculateSizeBlock)completionBlock;
    异步计算磁盘缓存大小
#pragma mark - Cache Paths

- (nullable NSString *)cachePathForKey:(nullable NSString *)key inPath:(nonnull NSString *)path;
    获得缓存文件路径，需要传入缓存根目录

- (nullable NSString *)defaultCachePathForKey:(nullable NSString *)key;
    获得默认的缓存文件路径

```
### SDWebImageDownloadToken
```objective-c
@property (nonatomic, strong, nullable) NSURL *url;
    下载地址
    
@property (nonatomic, strong, nullable) id downloadOperationCancelToken;
    用于取消下载任务的标识符
@property (nonatomic, weak, nullable) NSOperation<SDWebImageDownloaderOperationInterface> *downloadOperation;
```

### SDWebImageDownloader
```objective-c

@property (assign, nonatomic) BOOL shouldDecompressImages;
    将下载完成的或者缓存完成的图片解压，可以提高性能，但是会有内存损耗

@property (assign, nonatomic) NSInteger maxConcurrentDownloads;
    下载最大并发数，默认为6

@property (readonly, nonatomic) NSUInteger currentDownloadCount;
    当前正在下载的任务数
    
@property (assign, nonatomic) NSTimeInterval downloadTimeout;
    下载任务超时时间，默认15秒

@property (readonly, nonatomic, nonnull) NSURLSessionConfiguration *sessionConfiguration;
   NSURLSession创建的时候需要的配置

@property (assign, nonatomic) SDWebImageDownloaderExecutionOrder executionOrder;
    改变下载顺序，默认先进先出
    有队列和栈两种形式

@property (strong, nonatomic, nonnull) NSOperationQueue *downloadQueue;
    下载队列
@property (weak, nonatomic, nullable) NSOperation *lastAddedOperation;
    最新加入的下载操作
@property (assign, nonatomic, nullable) Class operationClass;
    实现下载的NSOperation类
@property (strong, nonatomic, nonnull) NSMutableDictionary<NSURL *, NSOperation<SDWebImageDownloaderOperationInterface> *> *URLOperations;
    下载操作的内存缓存
@property (strong, nonatomic, nullable) SDHTTPHeadersMutableDictionary *HTTPHeaders;
    发起下载的HTTP Header
@property (strong, nonatomic, nonnull) dispatch_semaphore_t operationsLock; // a lock to keep the access to `URLOperations` thread-safe
    操作Operation的锁
@property (strong, nonatomic, nonnull) dispatch_semaphore_t headersLock; // a lock to keep the access to `HTTPHeaders` thread-safe
    操作Header的锁

// The session in which data tasks will run
@property (strong, nonatomic) NSURLSession *session;
    网络配置

+ (nonnull instancetype)sharedDownloader;
    全局实例

@property (strong, nonatomic, nullable) NSURLCredential *urlCredential;
    设置请求的凭证，如用户名密码，或者证书

@property (strong, nonatomic, nullable) NSString *username;
    设置请求验证用户名

@property (strong, nonatomic, nullable) NSString *password;
    设置请求验证密码

@property (nonatomic, copy, nullable) SDWebImageDownloaderHeadersFilterBlock headersFilter;
    构造下载请求HTTP Header的回调。每一次请求都会被调用

- (nonnull instancetype)initWithSessionConfiguration:(nullable NSURLSessionConfiguration *)sessionConfiguration NS_DESIGNATED_INITIALIZER;
   根据NSURLSessionConfiguration创建实例

- (void)setValue:(nullable NSString *)value forHTTPHeaderField:(nullable NSString *)field;
    设置图片下载的Request的HTTP Header，每次请求都会被使用

- (nullable NSString *)valueForHTTPHeaderField:(nullable NSString *)field;
    获取HTTP Header中的某个值

- (void)setOperationClass:(nullable Class)operationClass;
    设置符合SDWebImageDownloaderOperationInterface协议的NSOperation来创建下载，默认为SDWebImageDownloaderOperation


- (nullable SDWebImageDownloadToken *)downloadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageDownloaderOptions)options
                                                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                                 completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock;
     根据给定URL来创建下载

- (void)cancel:(nullable SDWebImageDownloadToken *)token;
    取消下载

- (void)setSuspended:(BOOL)suspended;
    挂起下载队列

- (void)cancelAllDownloads;
    取消所有下载

- (void)createNewSessionWithConfiguration:(nonnull NSURLSessionConfiguration *)sessionConfiguration;
    根据NSURLSessionConfiguration创建一个新的NSURLSession

- (void)invalidateSessionAndCancel:(BOOL)cancelPendingOperations;
    取消所有任务，可以选择是否取消等待中的任务

```

## SDWebImageDownloaderOperation : NSOperation <SDWebImageDownloaderOperationInterface, SDWebImageOperation>
```objective-c

@property (strong, nonatomic, readonly, nullable) NSURLRequest *request;
    网络请求

@property (strong, nonatomic, readonly, nullable) NSURLSessionTask *dataTask;
    下载任务

@property (assign, nonatomic) BOOL shouldDecompressImages;
    是否解压图片
    
@property (nonatomic, strong, nullable) NSURLCredential *credential;
    请求验证
/**
 * The SDWebImageDownloaderOptions for the receiver.
 */
@property (assign, nonatomic, readonly) SDWebImageDownloaderOptions options;
    下载特性选项

@property (assign, nonatomic) NSInteger expectedSize;
    预期下载的大小

@property (strong, nonatomic, nullable) NSURLResponse *response;
    请求获取到的响应

- (nonnull instancetype)initWithRequest:(nullable NSURLRequest *)request
                              inSession:(nullable NSURLSession *)session
                                options:(SDWebImageDownloaderOptions)options NS_DESIGNATED_INITIALIZER;

- (nullable id)addHandlersForProgress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                            completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock;
    添加进度回调和完成回调，返回一个用于取消的Token
- (BOOL)cancel:(nullable id)token;
    取消当前操作

```