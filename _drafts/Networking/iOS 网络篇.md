## NSURLSession工作流程
##### NSURLSessionDataTask 发起一个POST HTTP请求

1. 创建一个NSSessionConfiguration
2. 用Configuration创建一个NSURLSession，设置缓存策略，delegate（Session即将结束，收到身份验证挑战，后台请求任务完成），Task所在线程
3. 创建一个NSURLRequest，设置请求类型为POST，构造HTTP Header，将参数放入HTTPBody
4. 用NSURLSession创建一个NSURLSessionDataTask，传入Request，设置delegate（如网络等待，身份挑战，数据接收进度，任务完成）
5. 调用 Task Resume方法，唤醒Task
6. 在身份验证回调中处理证书信任问题
7. 在接收到响应回调中处理是否继续接受数据
8. 在数据接收回调中处理数据，更新进度
9. 最后关闭Session

##### NSURLSessionStreamTask工作流程
发起一个请求，以流读写的方式传递数据，异步读写，双向通信的TCP连接
在回调中处理数据

##### NSURLSessionDownloadTask断点续传过程
1. 当任务发生错误中断的时候，Task会包含出错的Error信息，Error的Userinfo中有一个字段NSURLSessionDownloadTaskResumeData，其中就包含了ResumeData。
2. 手动调用cancelByProducingResumeData，中断DownloadTask，并获得ResumeData。
3. 在恢复下载是，使用ResumeData创建DownloadTask即可。

##### 后台下载功能

1. 配置系统权限Newsstand downloads
2. 创建一个SessionConfiguration，初始化的时候用backgroundSessionConfigurationWithIdentifier:方法，并且指定任务唯一标识
3. 设置Configuration的sessionSendsLaunchEvents为YES，使任务在完成的时候会唤起App
4. 启动任务
5. 当任务完成后，系统会唤起App，并会调用application:handleEventsForBackgroundURLSession:completionHandler:方法
6. 如果期间APP发生手动退出或者闪退，重新启动后可以再次获取到相同唯一标示的任务。

**1. 在project中开启后台下载权限**
**2. 初始化用于后台下载的Session，并获得唯一标识符**
**3. 当任务完成后会回调AppDelegate的后台任务完成，也会回调Session的任务完成会回调**

##### NSURLCache运作
只能缓存GET请求
1. 指定缓存内存容量大小、缓存硬盘容量大小来创建一个NSURLCache
2. 将指定的NSURLRequest和相关联的NSURLCachedResponse存储Cache，下次根据NSURLRequest来获取对应的NSURLCachedResponse。
3. NSURLCachedResponse对应的缓存策略
    1. 不限制缓存（磁盘，内存都可以）
    2. 只允许内存缓存
    3. 不允许缓存
4. 通过Category扩展，NSURLCache可以通过NSURLSessionDataTask来关联NSURLCachedResponse。

**1. 当request成功获得响应后，response会被存入hash表，关联key为request的内存地址。**
**2. 当request再次请求时，会先发起请求验证response的数据是否有效，如果有效则直接返回缓 存，如果过期则发起新的请求。**
**3. 不同的缓存策略会有不同的效果。**

##### NSURLProtocol

1. 在Session,或者全局中注册NSURLProtocol
2. 在NSURLSessionDataTask中设置调用NSURLProtocolClient协议实现后的方法
3. 当发起URL加载的时候，系统会在已经注册过的NSURLProtocol中寻找对应可以处理NSURLRequest的Class来处理。
4. NSURLProtocol开始处理NSURLRequest，重新构造NSURLRequest。
5. 在加载开始后，将NSURLRequest标记为已经执行，防止重复执行陷入循环。
6. NSURLProtocol只能拦截NSURLSession和UIView中的URL加载，他们在加载时都用的底层Socket连接，WKWebView属于WebKit，不走Socket，所以无法拦截。

**1. 可以注册为全局生效，或者关联的Session**
**2. 可以使用请求发起，获得响应，获得数据，加载完成，加载失败，身份挑战等回调。**
**3. 已处理过的Request需要标记为已完成，防止重复执行**

---
## AFNetworking工作流程

##### 发起一个POST HTTPS请求
1. 生成一个AFHTTPSessionManager，设置BaseURL
2. 配置RequestSerialization和ResponseSerialization。
3. RequestSerialization将生成Requst的Header（Content-Type，User-Agent，Accept-Language，）、HTTPBody，如是POST请求方式，将参数以NSData的形式放入HTTPBody中，如是GET请求，则将参数拼在URL上。
4. 根据Request来创建一个NSURLSessionDataTask，将Task放入一个并发数为1的线程队列中，同时创建一个独立的delegate放入Map中。
5. 当Task开始运行，发起HTTPS请求，每次收到数据都会用NSProgress来统计进度
6. 发生身份验证
  1. 验证类型为公钥验证，提取服务端trust中的公钥，比对本地公钥，然后将结果传回服务端
  2. 验证类型为证书验证，服务器trust与证书域名做匹配，如果设置了本地证书，则将证书加入到服务器trust中，然后将结果传回服务端。
  3. 也可以拒绝响应或取消验证。
7. 当得到Response的时候，ResponseSerialization会进行域名和状态码验证，验证失败会抛出Error
8. 验证成功后，ResponseSerialization进行responseObject的创建，期间的处理有转义成JSON，XML，PropertyList，解压PNG或JPG等。

**1. 创建一个HTTPSessionManager，配置BaseURL**
**2. 配置Request Serialization，Response Serialization**
**3. 调用Request Serialization来创建Request，配置Request的URL，将Params解析成NSData后存入Body。**
**4. 创建一个DataTask，并未DataTask创建一个独立的delegate，并缓存起来。**
**5. 当请求遇到身份挑战的时候，Task的delegate会处理身份挑战，如果是公钥验证，则从证书中获取公钥并与本地校验，通过则回传服务端；如果是证书验证，则先验证证书域名和有效期，成功后将本地证书信息加入到服务端证书中，回传服务端；也可以拒绝响应或取消挑战。**
**6. 当获取到服务端响应时，Response Serialization将先验证域名和状态码，通过后根据设置的不同Serialization来格式化响应数据供开发人员使用。**

---
### SDWebImageManager 获取图片
1. 先检测当前请求的URL是否已经失败过，如果没有设置失败后尝试，则流程结束
2. 先检测内存缓存是否有对应的URL缓存，再磁盘缓存中是否存在，如果有则调用**完成回调**，流程结束
3. 如果缓存不存在，则会创建图片下载进程。
4. 下载线程将接收到的图片数据拼接在一起，如果采用了渐进式的显示模式，则会每收到一次都拼接一次，并执行**完成回调**
5. 如果出现超时、无法连接等网络错误，则会将URL放入失败过的URL Map中，防止下次重复执行。
6. 下载成功后，将图片解压，将data数据分别存入内存缓存和磁盘缓存。
7. 执行**完成回调**，取得图片

**1. 先查询当前URL是否已经尝试失败过，如果没有设置失败后尝试，则结束**
**2. 检测内存缓存是否有对应url的缓存，再检测磁盘是否有对应url的缓存，如果有则返回**
**3. 如果没有缓存则创建任务开始下载**
**4. 如果设置了渐进式显示，则在每次获得数据是都会生成图片并返回**
**5. 如果请求失败了，则url会被记录到失败Hash表中。**
**6. 下载成功后将图片解压，将图片分别放入内存缓存和磁盘缓存**

---
### HTTPS证书中都包含哪些内容
1. 所有者的公用密钥
2. 所有者的专有名称
3. 颁发证书的 CA 的专有名称
4. 证书生效的起始日期
5. 证书的到期日期
6. X.509 中定义的证书数据格式的版本号。X.509 标准的最新版本为 V3，大多数证书都遵循此版本。
7. 序列号。这是颁发证书的 CA 所分配的唯一标识。序列号在颁发证书的 CA 中是唯一的：同一 CA 证书签署的两个证书不会具有相同的序列号。

