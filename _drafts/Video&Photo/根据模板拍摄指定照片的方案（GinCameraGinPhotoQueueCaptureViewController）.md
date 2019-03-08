根据模板拍摄指定照片的方案（GinCamera/GinPhotoQueueCaptureViewController）



DEMO地址

GinCamera https://github.com/ginhoor/GinCamera.git



这里只做方案思路的讲解，具体实现，请参考DEMO

实现功能：

模板功能：

用来提供例如示例图，教程地址，照片拍摄类型（有身份模式保证照片与模板一一对应、无身份模式可以让照片前移，同时将对应模板改为前位模板，保证队列中没有空元素）。

通过后端配置的指定json模板，来拍摄照片，通过模板为照片赋能，将照片与模板解耦。

从模板中的任意一环都可以进入拍摄状态