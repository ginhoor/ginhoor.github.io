背景介绍：在接入七牛SDK的时候，发现SDK没有批量上传图片的接口，业务又涉及到了上传进度统计，并且要求一次性的图片完整上传。

开始的时候打算用GCD，写着写着感觉扩展性不好，可读性不高，取消机制也不是很友好，线程暂停需要用信号量来管理，于是改成了NSBlockOperation来实现

.h文件
```objective-c
/**
 串行队列
 
 @param dataList 数据源
 @param opreationBlock 运行回调
 @param cancelBlock 中断回调
 @param completionBlock 完成回调
 @return 线程队列
 */
+ (NSOperationQueue *)startOperationOnSerialQueue:(NSArray *)dataList
                     opreationBlock:(void(^)(id data, NSOperationQueue *queue, NSInteger index, double totalProgress))opreationBlock
                        cancelBlock:(BOOL(^)(id data, NSInteger index))cancelBlock
                    completionBlock:(void(^)(BOOL success))completionBlock;

```
介绍下h文件，这个类方法可以创建一个线程池，串行队列的顺序由dataList数组源中的元素顺序决定，提供三种不同时机的回调，根据实际情况使用，opreationBlock是主要回调，相当于thread的main。

.m文件
```objective-c
+ (NSOperationQueue *)startOperationOnSerialQueue:(NSArray *)dataList
                     opreationBlock:(void(^)(id data, NSOperationQueue *queue, NSInteger index, double totalProgress))opreationBlock
                        cancelBlock:(BOOL(^)(id data, NSInteger index))cancelBlock
                    completionBlock:(void(^)(BOOL success))completionBlock
{
    // 创建一个线程队列，最大并发数为1
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    queue.maxConcurrentOperationCount = 1;
    
    // 根据数据源创建Operation
    __block NSBlockOperation *lastBlockOperation;
    [dataList enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        __block NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:^{
            // 如果设置了中断回调，优先检测中断回调是否触发
            if (cancelBlock) {
                BOOL cancelFlag = cancelBlock(obj,idx);
                //如果中断了
                if (cancelFlag) {
                    //取消所有线程
                    [queue cancelAllOperations];
                    // 触发一次完成回调
                    [queue addOperation:[NSBlockOperation blockOperationWithBlock:^{
                        if (completionBlock) {
                            completionBlock(NO);
                        }
                    }]];
                } else {
                    if (opreationBlock) {
                        opreationBlock(obj, queue, idx, idx*1.0f/dataList.count);
                    }
                }
            } else {
                if (opreationBlock) {
                    opreationBlock(obj, queue, idx, idx*1.0f/dataList.count);
                }
            }
        }];
        
        // 将当前线程设置为上一次线程的依赖，保证运行顺序
        if (lastBlockOperation) {
            [blockOperation addDependency:lastBlockOperation];
        }
        lastBlockOperation = blockOperation;
        [queue addOperation:blockOperation];
        if (idx == dataList.count-1) {
            //当最后一个线程运行完毕后，触发完成回调
            NSBlockOperation *completionOperation = [NSBlockOperation blockOperationWithBlock:^{
                if (completionBlock) {
                    completionBlock(YES);
                }
            }];
            [completionOperation addDependency:blockOperation];
            [queue addOperation:completionOperation];
        }
    }];
    
    return queue;
}
```
流程都有注释，主要逻辑就是将线程设置成单项依赖，然后在适当的时机执行中断判断，触发完成回调。到现在为止完成了同步操作的串行队列执行，但大家都知道，同步操作即使不用队列，也是串行执行的，所以这里要解决异步操作的串行队列执行，这就用到了NSOperationQueue的suspended功能，在执行异步操作前先暂停队列，等异步操作完成后，再继续队列。

具体实现参考
```objective-c
NSMutableArray *imageKeys = [NSMutableArray array];
    __block NSString *lastResultKey = nil;
    [GinOperationQueueManager startOperationOnSerialQueue:imageList opreationBlock:^(id data, NSOperationQueue *queue, NSInteger index, double totalProgress) {
        // 这里挂起队列
        queue.suspended = YES;
        [self uploadSingleImage:data filename:[CEQiNiuManager createJPGImageKey] progress:nil completion:^(QNResponseInfo *info, NSString *key, NSDictionary *resp, UIImage *image) {
            lastResultKey = key;
            if (!key) {
                if (completion) {
                    completion(NO, nil);
                }
            } else {
                if (progress) {
                    progress(key, totalProgress ,index);
                }
                [imageKeys addObject:key];
            }
            //执行完成后继续队列
            queue.suspended = NO;
        }];
    } cancelBlock:^BOOL(id data, NSInteger index) {
        return ![lastResultKey isNotBlank] && index != 0;
    } completionBlock:^(BOOL success) {
        if (completion) {
            completion(success, imageKeys.count!=0?imageKeys:nil);
        }
    }];
```

这里是源文件，供大家参考
[DEMO地址](https://github.com/ginhoor/GinOperationQueueManager)