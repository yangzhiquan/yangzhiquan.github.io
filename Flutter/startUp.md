#### Flutter应用从0到1

背景：从0开始开发一个云盘应用.

##### 编码规范.
    Flutter的Linter规则记录在工程一级目录的analysis_options.yaml文件中。
##### 应用的基础能力

0. 页面跳转
    使用官方`go_router`库管理


1. 通讯能力：与webview交换信息，与服务器交换信息，与原生层交换信息。

与webview通讯: 使用官方的`webview_flutter`，有执行js接口，有channel提供接口给js调用。（还没支持回复js消息，可以参考jsbridge自行扩展消息callback能力.）

与native通讯: 通常来说有ffi跟methodChannel两类，ffi有性能优势，但使用起来没有channel友好。
官方提供的`pigeon`能自动生成methodchannel代码，但是每个接口都会生成一大串代码。如果对代码量比较在意还是建议使用一个methodchannel，根据参数区分不同的通讯需求.

与服务器通讯：使用dio, 的确是个很优秀的网络库，官方也推荐.

##### 代码生成

    Flutter图片资源的调用代码，可以用一些第三方库(或自行实现)自动生成, 比如`flutter_gen_runner`.
    网络请求接口函数也是能够api参数自动生成，`json_request`


## TODO

腾讯文档的问题？

view数据分离.

统一路径管理.

(卡顿)计算管理.

人力问题 > 技术问题（）

`旧框架的问题`

`细节`

`数据连通` 

<!-- 1. 持续集成 -->