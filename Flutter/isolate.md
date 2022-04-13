## Dart与单线程模型

### CPU核心与线程
核心：可以独立运行程序指令的计算单元。
线程：操作系统进行运算调度的最小单位。

### 多任务

一个系统，要实现多任务能力，通常会设计Master-Worker模式。一个Master负责任务派发，多个Worker执行。

用多进程实现，主进程是Master，其他进程是Workder; 
多线程实现，主线程是Master，子线程是Workder; 

两者的区别：
多进程相对安全，进程直接相对独立，互不影响；
多线程相对快捷，线程之间共享内存；


### 任务类型

通常来说计算机的执行任务分两类：计算密集型，IO密集型

计算密集型: 需要cpu进行大量的计算，比如编解码，计算圆周率；这种情况单核CPU分多线程任务反而增加没必要的切换上下文的时间.
IO密集型: 网络、磁盘IO. 不涉及大量的cpu运算. （处于等待IO或者休眠的线程不消耗CPU资源）


### Dart的异步编程

Dart是单线程，基于`任务队列`跟`事件循环`实现了 Future、async、await、completer 异步执行方法，避免执行iO任务的时候阻塞线程。（任务队列又包含Event queue 事件队列和1个 MicroTask queue，每次循环都会优先把MicroTask执行完毕再执行一个EventTask）
对于CPU密集型任务，Dart提供了Isolate以及它的封装方法compute(一种在另一个线程上运行Dart代码的方法)。

    通常情况，微任务的使用场景比较少。Flutter 内部也在诸如手势识别、文本输入、滚动视图、保存页面效果等需要高优执行任务的场景用到了微任务。

    所以，一般需求下，异步任务我们使用优先级较低的 Event Queue。比如 IO、绘制、定时器等，都是通过事件队列驱动主线程来执行的。

    Dart 为 Event Queue 的任务提供了一层封装，叫做 Future。把一个函数体放入 Future 中，就完成了同步任务到异步任务的包装（类似于 iOS 中通过 GCD 将一个任务以同步、异步提交给某个队列）。Future 具备链式调用的能力，可以在异步执行完毕后执行其他任务（函数）。
    
    Future(() => print('task1'));



eg: 
```dart
completer:
    Future<String> sendData(data) {
        xxx.send(data);
        // 创建一个completer，先返回一个future
        _completer = Completer<String>();
        return _completer.future;
    }
    // 自行决定返回数据的时机
    _completer.complete('xxxx');
```
```dart
compute:
// 类的静态方法, 或者顶级方法 ( 不包再类里面的方法 
function computingIntensive( val ){
  // 一些很耗时的计算
  return res
}
/// 正常使用，接受一个方法跟一个参数.
var res = await compute( callback , val );
```

```dart
// 创建Future方式
Future() //默认Event队列
Future.microtask() //mocro队列
Future.sync() //立即执行
Future.value()
Future.delayed()
Future.error()
```

*Isolate是分离的运行线程,并且不和主线程的内存堆共享内存.这意味着不能访问主线程中的变量、函数. 没有竞争的可能性所以不需要锁，也就不用担心死锁的问题。通信唯一方式只能是通过Port进行。
*使用Isolate也不是没有代价的，创建Isolate，拷贝参数数据都有耗时，额外的内存开销也需要关注。
*LoadBalancer：线程池管理库.

### 最后, 为什么用单线程模型？
1. Flutter是UI框架，处理的任务大部分是IO密集型，使用单线程足够.
2. 不涉及线程管理，资源竞争。线程安全，框架实现复杂度大大降低，开发维护都简单.
3. 没有线程切换、加锁开销，简化数据结构和算法的实现，大部分场景下效率更快.