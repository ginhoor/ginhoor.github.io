## 列表优化

1. 列表元素高度动态计算会增加CPU消耗，可以进行缓存，减少计算量
2. 列表元素中的固定图片使用imageByName获取，系统会自动在内存中进行缓存。
3. 减少列表元素的个数和层级，复杂UI可以考虑通过CoreGraphics绘制
4. 减少透明View多层级使用，多个层级的透明View渲染会增加CPU消耗。

## 离屏渲染

  离屏渲染需要开辟一个新的缓存区进行渲染操作，然后进行上下文切换，将从当前屏幕切换到离屏，等离屏渲染结束后，再将缓存区的数据显示到当前屏幕，这又是一次上下文切换。
  离屏渲染的缓存区是有限的，大约为屏幕像素的两倍大小。
  离屏渲染的缓存在未使用超过100ms后会被回收。

  会触发离屏渲染的有：
    1. DrawRect（CPU渲染后再传入缓存区）
     使用CoreGraphics的绘制操作用的是CPU渲染，整个渲染过程由CPU在APP内同步完成，最后将位图交由GPU显示。

    2. 光栅化（layer.shouldRasterize ）
     光栅化后的内容，在对应layer及其sublayers没有发生变化的情况下，不会再重新渲染。
     开启后，记得设置cell.layer.rasterizationScale = [[UIScreen mainScreen] scale];
    3. 遮罩Mask（layout.masksToBounds）
     layer.cornerRadius，layer.borderWidth，layer.borderColor不会引起离屏渲染。
     layer.cornerRadius和layout.masksToBounds组合使用，会产生离屏渲染。

    4. 阴影Shadow（layout.shadow）
    5. EdgeAntialiasing（抗锯齿）
     对性能影响不大
    6. GroupOpacity（透明度）
     只要父层的alpha设置为1，则不会触发离屏渲染。比如列表中，Cell.contentView.alpha = 1
     对性能影响不大
    7. Core Text绘制（CPU渲染后再传入缓存区）
    8. 渐变

  使用建议：
  1. 尽量只用在需要重复使用的位图上，比如某个位图需要在动画中被重复使用，这个时候应当开启shouldRasterize进行光栅化。
  2. 尽量使用当前屏幕渲染。
  3. 虽然GPU的浮点运算比CPU强，CPU的渲染效率是不如离屏渲染的，但如果只是简单的效果，直接使用CPU渲染效果可能比离屏渲染的开销更小。
  4. 出现性能问题后先查看是CPU瓶颈还是GPU瓶颈，再决定用什么方案。
  5. 使用CAShapeLayer来制作圆角遮罩的方式，代替直接设置圆角。

  