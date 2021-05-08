#### HTTP网络

NSProxy，不同于NSObject, 它只实现了NSObject，是一个轻量的类，唯一的功能就是NSObject协议定义的消息转发。所以它通常用来hook某些对象。
一般会持有一个对象，hook它的消息调用后，通过消息转发给持有的对象处理.

- (void)forwardInvocation:(NSInvocation *)invocation;
- (nullable NSMethodSignature *)methodSignatureForSelector:(SEL)sel;




## allowInterveneNetwork 为 YES

#### QAPMNSURLSessionProxy 用于 hook sessionDelegate<NSURLSessionDelegate>，上报回调信息
1. sessionWithConfiguration:delegate 的方式创建

```objc
    [[QAPMHTTPMonitor shared] qapm_urlSessionTask:task totalBytesExpectedToSend:totalBytesExpectedToSend];
    [[QAPMHTTPMonitor shared] qapm_urlSessionTaskDidStop:task error:error];
    [[QAPMHTTPMonitor shared] qapm_urlSessionTask:task didFinishCollectingMetrics:metrics];
    [[QAPMHTTPMonitor shared] qapm_urlSessionTask:dataTask didReceiveResponse:response ts:[QAPMTimeHelper timestampInMiniseconds]];
```


#### NSURLSession (QAPMHook) hook 生成task的各种方法，替换 completionHandler，上报回调信息.
特别的由于旧系统的实现不一样，对 resume 进行hook 写了很长的一段

```objc
[[QAPMHTTPMonitor shared] qapm_urlSessionTaskDidStart:(NSURLSessionTask *)_self];
[[QAPMHTTPMonitor shared] qapm_urlSessionTaskDidStop:task error:error];
```




1. 在resume时注入qapm_urlSessionTaskDidStart，创建QAPMAppRequestInfo
NSURLSessiontask任务进行时，首先调用resume，然后会调用qapm_urlSessionTaskDidStart，创建一个QAPMAppRequestInfo的对象，放入requestUrlDict[task]的字典里，根据resume的参数，复制QAPMAppRequestInfo里面的属性

2. hook各种回调
而后等着系统回调，对QAPMAppRequestInfo进行补充，

3. urlSessionTaskDidStop

完成最终AppRequestInfo的收集，然后上传

这里参考了AFN的内容。