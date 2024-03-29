### 简介
flutter是一个UI框架，简单点说它是在native端的一个view上绘制界面、处理事件。
我们讨论移动端开发的时候，有几个指标是常常提起的FPS，CPU，内存和GPU性能。


### 好处
跨平台，省人力(熟练掌握的前提下)，

### 代价
运行flutter引擎需要额外的内存、cpu资源，包体积.
可能没办法享受到某些原生特性.
难以像原生语言那样充分利用GPU.


### 不足
flutter仅仅是UI框架，访问硬件、多媒体、存储以及浏览器应用生命周期等能力还是需要native。

### 横向对比（React Native

1. 资源占用
flutter并不大依赖原生，如果不涉及特定的功能，flutter业务逻辑可以完全不与原生通讯。
相比之下，RN的JS和原生之间的通讯会在序列化和反序列化方面消耗资源（cpu/内存/电量
相比之下，RN的运行没能像flutter那样更好得使用GPU资源。

2. 性能
 dart是静态语言，相比动态语言有更高的执行效率。同时Flutter又支持JIT模式, 兼顾开发效率；
 动画性能消耗方面，RN应该还是相对不占优。

3. UI
RN依赖native组件，各端有各自的风格；，flutter是自己绘制，全端统一风格。


 ### React Native简述

浏览器展示网页：浏览器按规定实现了一套组件。前端代码用html描述页面有什么组件，css描述组件的样子。浏览器解析html/css之后展示出界面来，解析js代码接收/响应事件，调整界面。
RN跟浏览器很类似，它也有一套UI组件，解析前端代码之后，转发到native层组合/调整页面。

React Native是依靠JS Engine解析执行js代码的（iOS上是JavaScriptCore）. 虽然用JavaScriptCore作为JS的解析引擎，但自己实现了一套通讯机制，没有JavaScriptCore的情况下也可以用webview代替进行js解析。

模块配置表：Objective-C 和 JavaScript 两端都保存了一份配置表，里面标记了所有 Objective-C 暴露给 JavaScript 的模块和方法。这样，无论是哪一方调用另一方的方法，实际上传递的数据只有 ModuleId、MethodId 和 Arguments
这三个元素，它们分别表示类、方法和方法参数，当 Objective-C 接收到这三个值后，就可以通过 runtime 唯一确定要调用的是哪个函数，然后调用这个函数。

OC要告知JS它有什么模块，模块内的方法，参数。JS知道后才可能去调用这些方法。
OC端和JS端分别各有一个bridge，两个bridge都保存了同样一份模块配置表，JS调用OC模块方法时，通过bridge里的配置表把模块方法转为模块ID和方法ID传给OC，OC通过bridge的模块配置表找到对应的方法执行。
js调用native时会把callback函数存起来，同时生成个id传给native，这样native回传时带上id，js端就可以执行对应的callback函数了。


