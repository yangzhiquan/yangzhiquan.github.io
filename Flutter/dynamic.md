# Flutter与动态化


### 前言

编程语言要达到可运行的目的需要经过编译，一般地来说，编译模式分为两类：JIT 和 AOT。

JIT全称 Just In Time(即时编译），典型的例子就是 v8，它可以即时编译并运行 JavaScript。所以你只需要输入源代码字符串，v8就可以帮你编译并运行代码。通常来说，支持 JIT的语言一般能够支持自省函数（eval），在运行时动态地执行代码。

AOT全称 Ahead Of Time（事前编译），典型的例子就是 C/C++，LLVM或 GCC通过编译并生成 C/C++的二进制代码，然后这些二进制通过用户安装并取得执行权限后才可以通过进程加载执行。

### Flutter的编译模式
Script：同 Dart Script模式一致，虽然 Flutter支持，但暂未看到使用，毕竟影响启动速度。
Script Snapshot：同 Dart Script Snapshot一致，同样支持但未使用，Flutter有大量的视图渲染逻辑，纯 JIT模式影响执行速度。
Kernel Snapshot：Dart的 bytecode 模式，与 Application Snapshot不同，bytecode模式是不区分架构的。 Kernel Snapshot在 Flutter项目内也叫 Core Snapshot。bytecode模式可以归类为 AOT编译。
Core JIT：Dart的一种二进制模式，将指令代码和 heap数据打包成文件，然后在 vm和 isolate启动时载入，直接标记内存可执行，可以说这是一种 AOT模式。Core JIT也被叫做 AOTBlob
AOT Assembly: 即 Dart的 AOT模式。直接生成汇编源代码文件，由各平台自行汇编。


[Introduction to Dart VM](https://mrale.ph/dartvm/)

Flutter的核心还是跨平台而不是动态化，但这两者都是目前业界的诉求，因此才会有那么多的动态化方案产生。
Flutter框架的开发语言是Dart，实现动态化需要关注的是Dart语言的编译、运行过程.

- `AST`: 编译器把开发编写的业务代码跟FlutterSDK解析为一个抽象语法树（内存对象）
- `dill`、`bin`: AST对象序列化存到磁盘的文件类型; eg:app.dill，或者Debug阶段的产物(kernel_blob.bin) 
- `Dart Kernel`: AST文件的数据格式，能够被Dart VM解释执行；也是dart2js或其他转换的中间语言.
- `IL`: Intermediate Language, 要生成AOT产物，编译器会加载dill文件，编译成类似字节码的中间语言。

### 动态法方案

##### 1.自定义模板代码

借鉴RN的思路，选一门可以在客户端执行的解析行语言，自定义一套模板跟目标语言的UI组件一一对应。比如采用js语言，再用json格式来定义模板语言。

基于这个思路的实现：
58的[fair](https://github.com/wuba/fair)

    优点：从RN的发展来看，这个方向是可取的.
    缺点：代码执行至少多一层解析。另外，都是用前端生态了，为何不直接用RN？？

##### 2.替换开发语言
[mxflutter](https://github.com/mxflutter/mxflutter)

代替Dart生成Widget树的职责。
用JS完整实现一遍Flutter SDK内置在工程中，JSCore或V8引擎运行编写的业务JS代码，生成一个WidgetTree，并序列化为json文件。Dart端加载该文件，转换为真正的Widget-Tree并进行渲染。

问题点：

    TS/JS->JSWidget->json->FlutterWidget链路较长，问题定位比较困难；生成WidgetTree效率就相对要低，再加上数据及事件通知需要反复经JS-Dart通道进行通讯等因素，都会导致UI渲染效率低下。


##### 3. 基于AST导出DSL


美团的MTFlutter: [Flap](https://tech.meituan.com/2020/06/23/meituan-flutter-flap.html)

原理是把Dart代码用Analyzer工具转为AST，简化保存为json文件。
终端有一个`解释器`，递归访问json中的节点，然后将json中的字符串映射到Dart代码中的函数，然后调用函数的万能方法Function.apply(),完成对应代码指令的执行。

    eg: 我们看"#FontWeight.normal"这个节点，从字符串到函数的映射就是{"#FontWeight.normal":()=>FontWeight.normal}。在访问"#FontWeight.normal"命名的这个节点时，只需调用:()=>FontWeight.normal这个函数的Apply()方法即可创建出对应的Dart对象。当然完整的实现还有更复杂的自定义类的解释要支持继承、混入等语法， 变量要支持作用域的管理等。

优点：

    前两个思路都是比较表层的动态化，对Dart开发有很大的影响 (动态化跟目标代码语言不同)。而从AST层往下的动态化则可以让上层Dart开发者在编码阶段无感知。

问题点：

    由于AST未经过编译，每个代码文件生成的AST中都只包含当前文件的信息，缺乏代码精确的数据类型、代码上下文及依赖关系等关键信息。因此在解释过程中经常会遇到类型推断上的困难，在处理比较复杂的继承关系或隐式依赖时，因缺乏对应信息而找不到对应的符号以完成执行。MTFlutter解决这一问题的方法是限制Dart语法的使用。

##### 4. 基于Dill文件

UC的[Aion](https://mp.weixin.qq.com/s/mPkx9b07xCkokxxbGc7grA)

需要动态化的代码编译为Dill文件后，通过在AOT模式下增加对KBC 解释器的支持，实现KBC与AOT在混合模式下运行。，直接用DartVM将这部分解释执行；

问题点： 

    Flutter新版本已经裁剪掉了KBC解释器代码，为适应Flutter后续版本的升级和变化，需要自行对KBC解释器代码进行维护，维护的成本随着Flutter版本的不断升级也将不断升高。而且内置一个解释器也增加不少包体积。

##### 5. 基于IL指令集

基于Dart的中间语言(IL, Intermediate Language)，参考并定制Dart VM本身的C层实现，在Dart层设计了一套指令集和栈帧式虚拟机，在运行时，将IL转化为指令集逐条执行。

实现包含两个关键要素：

① 生成IL

Flutter&Dart工具链：编译Flutter&Dart源码，得到中间语言IL
IL二进制格式的读写：将IL输出为二进制格式，提升读取效率
② 执行IL

指令集：作为IL的内存数据表示
栈帧式虚拟机：执行指令，对栈元素进行精准控制
代理生成器：自动生成Flutter&Dart核心库的代理实现