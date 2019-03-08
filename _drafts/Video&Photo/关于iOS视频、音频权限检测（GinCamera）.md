关于iOS视频、音频权限检测（GinCamera/GinAVCaptureManager）

先附上DEMO
[GinCamera](https://github.com/ginhoor/GinCamera)
***

```objective-c
/**
 检查麦克风授权
 */
(BOOL)checkAudioAuthorization
{
    return [self checkAuthorizationStatus:AVMediaTypeAudio];
}


/**
 检测摄像头授权
 */
(BOOL)checkVideoAuthorization
{
    return [self checkAuthorizationStatus:AVMediaTypeVideo];
}

+ (BOOL)checkAuthorizationStatus:(AVMediaType)mediaType
{
    AVAuthorizationStatus authorStatus = [AVCaptureDevice authorizationStatusForMediaType:mediaType];
    if (authorStatus == AVAuthorizationStatusRestricted ||
        authorStatus == AVAuthorizationStatusDenied) {
        return NO;
    }
    return YES;
}
```
```objective-c
/**
 请求设备权限授权

 @param mediaType 设备类型
 @param successBlock 成功回调
 @param failtureBlock 失败回调
 */
- (void)requestCaptureAuthorizationByMediaType:(AVMediaType)mediaType successBlock:(void(^)(void))successBlock failture:(void(^)(void))failtureBlock
{
    //  检测授权
    switch ([AVCaptureDevice authorizationStatusForMediaType:mediaType]) {
        //  已授权，可使用
        //  The client is authorized to access the hardware supporting a media type.
        case AVAuthorizationStatusAuthorized: {
            if (successBlock) {
                successBlock();
            }
            break;
        }
        //  未进行授权选择
        //  Indicates that the user has not yet made a choice regarding whether the client can access the hardware.
        case AVAuthorizationStatusNotDetermined: {
            // 再次请求授权
            [AVCaptureDevice requestAccessForMediaType:mediaType completionHandler:^(BOOL granted) {
                //用户授权成功
                if (granted) {
                    if (successBlock) {
                        successBlock();
                    }
                } else {
                    //用户拒绝授权
                    if (failtureBlock) {
                        failtureBlock();
                    }
                }
            }];
            break;
        }
        //用户拒绝授权/未授权
        default: {
            if (failtureBlock) {
                failtureBlock();
            }
            break;
        }
    }
}
```

