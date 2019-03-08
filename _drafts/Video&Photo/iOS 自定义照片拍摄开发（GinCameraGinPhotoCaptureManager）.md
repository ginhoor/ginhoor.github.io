# iOS 自定义照片拍摄开发（GinCamera/GinPhotoCaptureManager）

先附上DEMO
[GinCamera] (https://github.com/ginhoor/GinCamera)

具体的详细创建过程请直接下载demo查阅，本教程主要是为了补充基础概念和一些注意事项

---

### 将要用到的类有：
AVCaptureDevice
AVCaptureDeviceInput
AVCaptureMetadataOutput
AVCapturePhotoOutput
AVCaptureSession
AVCaptureVideoPreviewLayer

### 下面将这些类一个简单介绍下

AVCaptureDevice 用于获取硬件设备，所有拍摄操作都将基于此硬件

###### 拍摄需要获取的是AVMediaTypeVideo类型设备
```objective-c
self.device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
[self.session setSessionPreset:AVCaptureSessionPresetPhoto]
```
当页面页面不再需要显示预览图的时候，请stopRunning session，保持session开启会消耗大量系统资源


###### AVCaptureDeviceInput  获取硬件的输入流
输入流中包含的是摄像头采集到的数据，我们可以通过这个类来处理摄像头的数据。
```objective-c
self.input = [AVCaptureDeviceInput deviceInputWithDevice:self.device error:nil];
```
###### AVCapturePhotoOutput 生成照片输出流
我们将输入流包装处理后，以输出流的形式来输出，最终生成照片
```
self.captureOutput = [[AVCapturePhotoOutput alloc] init];
```
###### AVCaptureSession 会话，关联输入流和输出流，生成预览层
```objective-c
self.session = [[AVCaptureSession alloc]init];

[self.session setSessionPreset:AVCaptureSessionPresetPhoto];

if ([self.session canAddInput:self.input]) {
	[self.session addInput:self.input];
}

if ([self.session canAddOutput:self.captureOutput]) {
	[self.session addOutput:self.captureOutput];
}
```

###### AVCaptureVideoPreviewLayer 视频拍摄阶段的预览层
提供取景画面的展示
```objective-c
self.preview = [AVCaptureVideoPreviewLayer layerWithSession:self.session];
```

### 下面介绍一些拍照常用的方案。
---
###### 点击取景画面，手动对焦画面
```objective-c
/**
 设置手动对焦

 @param focusMode 对焦模式
 @param exposureMode 曝光
 @param point 对焦点
 */
- (void)focusWithMode:(AVCaptureFocusMode)focusMode
         exposureMode:(AVCaptureExposureMode)exposureMode
              atPoint:(CGPoint)point
{
    // 安全的加锁赋值过程
    [self changeDevice:self.device property:^(AVCaptureDevice *device) {
        // 设置聚焦
        if ([device isFocusModeSupported:focusMode]) {
            [device setFocusMode:focusMode];
        }
        if ([device isFocusPointOfInterestSupported]) {
            [device setFocusPointOfInterest:point];
        }
        // 设置曝光
        if ([device isExposureModeSupported:exposureMode]) {
            [device setExposureMode:exposureMode];
        }
        if ([device isExposurePointOfInterestSupported]) {
            [device setExposurePointOfInterest:point];
        }
    }];
}
```

这里需要注意的是，当你不需要手动对焦的时候，记得将FocusMode和ExposureMode恢复到自动状态，防止出现非预期的情况。
```objective-c
- (void)setupDevice
{
    [self changeDevice:self.device property:^(AVCaptureDevice *device) {
        //  开启自动持续对焦
        if ([device isFocusModeSupported:AVCaptureFocusModeContinuousAutoFocus]) {
            [device setFocusMode:AVCaptureFocusModeContinuousAutoFocus];
        }
        // 开启自动曝光
        if ([device isExposureModeSupported:AVCaptureExposureModeContinuousAutoExposure]) {
            [device setExposureMode:AVCaptureExposureModeContinuousAutoExposure];
        }
    }];
}

```
---
###### 摄像头切换以及闪光灯功能
```objective-c
- (void)switchVideoDevice:(AVCaptureSession *)session completion:(void (^)(AVCaptureDeviceInput *, AVCaptureDevicePosition, AVCaptureDevicePosition))completion
{
    __weak typeof(self) _WeakSelf = self;
    [super switchVideoDevice:session completion:^(AVCaptureDeviceInput *newInput, AVCaptureDevicePosition position, AVCaptureDevicePosition newPosition) {
        
        if (newPosition == AVCaptureDevicePositionBack) {
            // 后置摄像头
            if ([[_WeakSelf.captureOutput supportedFlashModes] containsObject:@(_WeakSelf.lastFlashMode)]) {
                _WeakSelf.photoSettings.flashMode = _WeakSelf.lastFlashMode;
            }
        } else {
            // 前置摄像头
            if ([[_WeakSelf.captureOutput supportedFlashModes] containsObject:@(AVCaptureFlashModeOff)]) {
                // 前置摄像头不支持闪光灯
                _WeakSelf.lastFlashMode = _WeakSelf.photoSettings.flashMode;
                _WeakSelf.photoSettings.flashMode = AVCaptureFlashModeOff;
            }
        }
        if (completion) {
            completion(newInput, position, newPosition);
        }
    }];
}
```
```objective-c
- (void)switchVideoDevice:(AVCaptureSession *)session completion:(void(^)(AVCaptureDeviceInput *newInput, AVCaptureDevicePosition position, AVCaptureDevicePosition newPosition))completion
{
    AVCaptureDevicePosition position = -1;
    AVCaptureDevice *newCamera = nil;
    AVCaptureDeviceInput *newInput = nil;
    AVCaptureDevicePosition newPosition = -1;
    
    NSArray *inputs = session.inputs;
    for (AVCaptureDeviceInput *input in inputs) {
        
        AVCaptureDevice *device = input.device;
        if ([device hasMediaType:AVMediaTypeVideo]) {
            position = device.position;
            newCamera = nil;
            newInput = nil;
            if (position == AVCaptureDevicePositionFront) {
                // 切换为后置
                newCamera = [self videoDeviceByPosition:AVCaptureDevicePositionBack];
                newPosition = AVCaptureDevicePositionBack;
            } else {
                // 切换为前置，前置摄像头不支持闪光灯
                newCamera = [self videoDeviceByPosition:AVCaptureDevicePositionFront];
                newPosition = AVCaptureDevicePositionFront;
            }
            
            newInput = [AVCaptureDeviceInput deviceInputWithDevice:newCamera error:nil];
            [session beginConfiguration];
            [session removeInput:input];
            [session addInput:newInput];
            [session commitConfiguration];
            break;
        }
    }
    if (completion) {
        completion(newInput, position, newPosition);
    }
}
```

这里需要特别注意的是，前置摄像头是没有闪光灯功能的，如果切换到前置摄像头，同时设置上开启闪光灯，会引起Crash。如果你的拍摄方案有保留上次拍照的设置配置，记得留意这点。