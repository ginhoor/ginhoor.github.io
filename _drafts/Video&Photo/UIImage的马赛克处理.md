马赛克的原理：将原有图一个像素点的颜色扩散到周围的像素点，使多个像素点的颜色都显示相同颜色，相当于分辨率下降，图像就会变得模糊不清。

如下代码
每个像素为8位颜色值，四个通道，分别表示ARGB（透明度、红、绿、蓝）。
对图片进行逐行扫描，如level为10，拷贝出坐标（0，0）点的颜色，将坐标（0，0）至坐标（10，10）都使用这个坐标（0，0）点的颜色，这样就达到了马赛克的效果

---
下面这段代码也是找资料的时候发现的，写的比较清楚，大家可以直接用
```objective-c
#pragma mark- 马赛克
//每个颜色值8bit
#define kBitsPerComponent (8)
#define kBitsPerPixel (32)
//每一行的像素点占用的字节数，每个像素点的ARGB四个通道各占8个bit
#define kPixelChannelCount (4)

- (UIImage *)transToMosaicImageByblockLevel:(NSUInteger)level
{
    // 获取BitmapData
    
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    CGImageRef imgRef = self.CGImage;
    CGFloat width = CGImageGetWidth(imgRef);
    CGFloat height = CGImageGetHeight(imgRef);
    CGContextRef context = CGBitmapContextCreate (nil,
                                                  width,
                                                  height,
                                                  //每个颜色值8bit
                                                  kBitsPerComponent,
                                                  //每一行的像素点占用的字节数，每个像素点的ARGB四个通道各占8个bit
                                                  width*kPixelChannelCount,
                                                  colorSpace,
                                                  kCGImageAlphaPremultipliedLast);
    CGContextDrawImage(context, CGRectMake(0, 0, width, height), imgRef);
    unsigned char *bitmapData = CGBitmapContextGetData (context);
    
    // 这里把BitmapData进行马赛克转换,就是用一个点的颜色填充一个level*level的正方形
    unsigned char pixel[kPixelChannelCount] = {0};
    NSUInteger index,preIndex;
    for (NSUInteger i = 0; i < height - 1 ; i++) {
        for (NSUInteger j = 0; j < width - 1; j++) {
            index = i * width + j;
            if (i % level == 0) {
                if (j % level == 0) {
                    memcpy(pixel, bitmapData + kPixelChannelCount*index, kPixelChannelCount);
                }else{
                    memcpy(bitmapData + kPixelChannelCount*index, pixel, kPixelChannelCount);
                }
            } else {
                preIndex = (i-1)*width +j;
                memcpy(bitmapData + kPixelChannelCount*index, bitmapData + kPixelChannelCount*preIndex, kPixelChannelCount);
            }
        }
    }
    
    NSInteger dataLength = width*height * kPixelChannelCount;
    CGDataProviderRef provider = CGDataProviderCreateWithData(nil, bitmapData, dataLength, nil);
    // 创建要输出的图像
    CGImageRef mosaicImageRef = CGImageCreate(width, height,
                                              kBitsPerComponent,
                                              kBitsPerPixel,
                                              width*kPixelChannelCount ,
                                              colorSpace,
                                              kCGBitmapByteOrderDefault,
                                              provider,
                                              nil, NO,
                                              kCGRenderingIntentDefault);
    CGContextRef outputContext = CGBitmapContextCreate(nil,
                                                       width,
                                                       height,
                                                       kBitsPerComponent,
                                                       width*kPixelChannelCount,
                                                       colorSpace,
                                                       kCGImageAlphaPremultipliedLast);
    CGContextDrawImage(outputContext, CGRectMake(0.0f, 0.0f, width, height), mosaicImageRef);
    CGImageRef resultImageRef = CGBitmapContextCreateImage(outputContext);
    UIImage *resultImage = nil;
    
    resultImage = [UIImage imageWithCGImage:resultImageRef];
    //释放
    if (resultImageRef) {
        CFRelease(resultImageRef);
    }
    if (mosaicImageRef) {
        CFRelease(mosaicImageRef);
    }
    if (colorSpace) {
        CGColorSpaceRelease(colorSpace);
    }
    if (provider) {
        CGDataProviderRelease(provider);
    }
    if (context) {
        CGContextRelease(context);
    }
    if (outputContext) {
        CGContextRelease(outputContext);
    }
    return resultImage;
}
```