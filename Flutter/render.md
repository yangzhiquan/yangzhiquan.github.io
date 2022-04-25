

#### Native侧的载体`FlutterViewController`:

- `layerClass`: CALayer(模拟器)；CAMetalLayer(支持METAL)；CAEAGLLayer(默认)
- 界面绘制: takeScreenshot函数获取Flutter当前界面的光栅图
- `TaskRunner`: platform(与native通讯,native主线程), ui(dart代码), gpu, io; 
- 无障碍支持: ensureSemanticsEnabled
- `native`方法绑定: `RegisterNatives`方法绑定的几个函数式两端通讯基础(Window_defaultRouteName、Window_scheduleFrame、Window_sendPlatformMessage、Window_respondToPlatformMessage、Window_render、Window_updateSemantics、Window_setIsolateDebugName...)
- `手势处理`: native会把event透传到flutter去派发
...

#### Flutter的UI绘制流程
![event](./asset/pipeline.webp)
用户输入信号(滑动、点击) > 驱动视图更新 > 触发动画进度 > build抽象视图数据 > 布局、绘制、合成（渲染过程的三个步骤），> 光栅化处理成像素填充数据

##### 视图数据结构：
Flutter的视图数据抽象分为3部分，分别是Widget、Element、RenderObject

- `Widget`: 布局、样式等配置信息
- 