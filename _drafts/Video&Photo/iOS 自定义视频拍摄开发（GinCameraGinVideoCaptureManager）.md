### iOS 自定义视频拍摄开发（GinCamera/GinVideoCaptureManager）

先附上DEMO
[GinCamera] (https://github.com/ginhoor/GinCamera)

视频拍摄和照片拍摄差不多，只是数据方面多了一个音频。

同样具体的详细创建过程请直接下载demo查阅

---
### 主要对象
```objective-c
@property (strong, nonatomic) AVCaptureSession *captureSession;
/**
 视频设备
 */
@property (strong, nonatomic) AVCaptureDevice *captureDevice;
@property (strong, nonatomic) AVCaptureDeviceInput *captureDeviceInput;
/**
 音频设备
 */
@property (strong, nonatomic) AVCaptureDevice *audioDevice;
@property (strong, nonatomic) AVCaptureDeviceInput *audioDeviceInput;
/**
 视频输出流
 */
@property (strong, nonatomic) AVCaptureMovieFileOutput *captureMovieFileOutput;
/**
 预览层
 */
@property (strong, nonatomic) AVCaptureVideoPreviewLayer *captureVideoPreviewLayer;
```

视频拍摄与拍照不同，需要先设定视频的存放地址

```objective-c
NSURL *fileUrl = [NSURL fileURLWithPath:filePath];
self.videoFilePath = filePath;
[self.captureMovieFileOutput startRecordingToOutputFileURL:fileUrl recordingDelegate:self];
```

视频拍摄需要同时获得摄像头权限与麦克风权限，常见到有人问为什么开了摄像头权限，但是取景画面不出来，这个时候请检查麦克风权限。


### 开启视频防抖功能
```objective-c
        AVCaptureConnection *captureConnection = [self.captureMovieFileOutput connectionWithMediaType:AVMediaTypeVideo];
        // 开启视频防抖
        if ([captureConnection isVideoStabilizationSupported]) {
            captureConnection.preferredVideoStabilizationMode = AVCaptureVideoStabilizationModeAuto;
        }
```


### 检测视频区域发生动态变化
通过监听AVCaptureDeviceSubjectAreaDidChangeNotification通知，可以获得回调
视频合成方法

###  获得指定视频的预览图
```objective-c
/**
 获得指定视频的预览图

 @param filePath 视频地址
 @return 预览图
 */+ (UIImage *)getVideoPreViewImage:(NSString *)filePath
{
    AVURLAsset *asset = [[AVURLAsset alloc] initWithURL:[NSURL fileURLWithPath:filePath] options:nil];
    
    AVAssetImageGenerator *generator = [[AVAssetImageGenerator alloc] initWithAsset:asset];
    generator.appliesPreferredTrackTransform = YES;
    CMTime time = CMTimeMakeWithSeconds(0.0, 600);
    CMTime actualTime;
    NSError *error = nil;
    CGImageRef image = [generator copyCGImageAtTime:time actualTime:&actualTime error:&error];
    if (error) {
        NSLog(@"get preview image failed!! error：%@",error);
        CGImageRelease(image);
        return nil;
    }
    UIImage *videoImage = [[UIImage alloc] initWithCGImage:image];
    CGImageRelease(image);
    return videoImage;
}
```

### 多个视频合成
```objective-c
/**
 视频合成

 @param videoFilePathList 视频地址
 @param outputPath 输出地址
 @param presetName 分辨率，默认：AVAssetExportPreset640x480
 @param outputFileType 输出格式，默认：AVFileTypeMPEG4
 @param completion 完成回调
 */

+ (void)mergeAndExportVideos:(NSArray <NSString *> *)videoFilePathList
                  outputPath:(NSString *)outputPath
                  presetName:(NSString *)presetName
              outputFileType:(NSString *)outputFileType
                  completion:(void(^)(BOOL success))completion
{
    if (videoFilePathList.count == 0) {
        return;
    }

    NSLog(@"videoFilePathList--->%@",videoFilePathList);
    
    AVMutableComposition *mixComposition = [AVMutableComposition composition];
    
    AVMutableCompositionTrack *audioTrack = [mixComposition addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:kCMPersistentTrackID_Invalid];
    
    AVMutableCompositionTrack *videoTrack = [mixComposition addMutableTrackWithMediaType:AVMediaTypeVideo preferredTrackID:kCMPersistentTrackID_Invalid];
    
    __block CMTime totalDuration = kCMTimeZero;
    
    [videoFilePathList enumerateObjectsUsingBlock:^(NSString * _Nonnull filePath, NSUInteger idx, BOOL * _Nonnull stop) {
        
        AVURLAsset *asset = [AVURLAsset assetWithURL:[NSURL fileURLWithPath:filePath]];
        
        NSError *audioError = nil;
        // 获取AVAsset中的音频
        AVAssetTrack *assetAudioTrack = [[asset tracksWithMediaType:AVMediaTypeAudio] firstObject];
        // 向通道内加入音频
        BOOL audioFlag = [audioTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, asset.duration)
                                             ofTrack:assetAudioTrack
                                              atTime:totalDuration
                                               error:&audioError];
        if (!audioFlag) {
            NSLog(@"audioTrack insert error:%@,%d",audioError,audioFlag);
        }
        
        // 向通道内加入视频
        NSError *videoError = nil;
        AVAssetTrack *assetVideoTrack = [[asset tracksWithMediaType:AVMediaTypeVideo] firstObject];
        [videoTrack setPreferredTransform:assetVideoTrack.preferredTransform];
        
        BOOL videoFlag = [videoTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, asset.duration)
                                             ofTrack:assetVideoTrack
                                              atTime:totalDuration
                                               error:&videoError];
        
        if (!videoFlag) {
            NSLog(@"videoTrack insert error:%@,%d",videoError,videoFlag);
        }
        totalDuration = CMTimeAdd(totalDuration, asset.duration);
    }];

    NSURL *mergeFileURL = [NSURL fileURLWithPath:outputPath];
    // 分辨率
    if (![presetName isNotBlank]) {
        presetName = AVAssetExportPreset640x480;
    }
    AVAssetExportSession *exporter = [[AVAssetExportSession alloc] initWithAsset:mixComposition presetName:presetName];
    exporter.outputURL = mergeFileURL;
    // 转出格式
    if (![outputFileType isNotBlank]) {
        outputFileType = AVFileTypeMPEG4;
    }
    exporter.outputFileType = outputFileType;
    exporter.shouldOptimizeForNetworkUse = YES;
    [exporter exportAsynchronouslyWithCompletionHandler:^{
        if (exporter.error) {
            NSLog(@"AVAssetExportSession Error：%@",exporter.error);
            if (completion) {
                completion(NO);
            }
            return;
        }
        // 导出状态为完成
        if ([exporter status] == AVAssetExportSessionStatusCompleted) {
            NSLog(@"视频合并完成--->%@",mergeFileURL);

            if (completion) {
                completion(YES);
            }
        } else {
            NSLog(@"AVAssetExportSession 当前压缩进度:%f",exporter.progress);
        }
    }];
}
```