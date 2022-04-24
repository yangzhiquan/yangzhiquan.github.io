## Flutter引擎启动过程

### FlutterViewController
FlutterViewController的FlutterView是Flutter在原生框架载体类.
它实现了两个协议 `FlutterTextureRegistry` 与 `FlutterPluginRegistry`;
PluginRegistry提供于flutter与原生数据交互能力，TextureRegistry提供flutter外接native纹理的能力.


初始化FlutterViewController需要一个FlutterEngine，它便是Flutter框架的运作核心；
它的作用包括了：

- Dart VM
- 各种keyEvent、lifecycle等等channel
