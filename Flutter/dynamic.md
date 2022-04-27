# Flutter动态化

### 前言
[Introduction to Dart VM](https://mrale.ph/dartvm/)

Flutter框架的开发语言是Dart，实现动态化需要关注的是Dart语言的编译、运行过程.

- `AST`: 编译器把开发编写的业务代码跟FlutterSDK解析为一个抽象语法树（内存对象）
- `dill`、`bin`: AST对象序列化存到磁盘的文件类型; eg:app.dill，或者Debug阶段的产物(kernel_blob.bin) 
- `Dart Kernel`: AST文件的数据格式，能够被Dart VM解释执行；也是dart2js或其他转换的中间语言.
- `IL`: Intermediate Language, 要生成AOT产物，编译器会加载dill文件，编译成类似字节码的中间语言。

### 动态法方案

1.自定义模板代码

借鉴RN的思路，选一门可以在客户端执行的解析行语言，自定义一套模板跟目标语言的UI组件一一对应。比如采用js语言，再用json格式来定义模板语言。

示例：[fair](https://github.com/wuba/fair)

    优点：从RN的发展来看，这套方案是可行的.
    缺点：代码执行至少多一层解析。另外，都是用前端生态了，为何不直接用RN？？


