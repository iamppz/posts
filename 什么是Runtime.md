前几天看到一位朋友在帖子里讲到Runtime就是一套API，感觉有些片面，在这里整理一下我的理解。

Runtime在.Net中叫CLR，在Java中叫JRE。

以CLR（公共语言运行时）举例，CLR是微软对CLI的实现，[CLR运行一种被称为CIL的语言的字节码](https://zh.wikipedia.org/wiki/%E9%80%9A%E7%94%A8%E8%AA%9E%E8%A8%80%E9%81%8B%E8%A1%8C%E5%BA%AB)。编译器将C#、F#等语言的代码编译成CIL，程序运行的时候CLR再通过JIT将CIL编译成机器码。

CLR提供以几个主要功能：
- 基类库支持 Base Class Library Support
  <br>
  基类库支持即BCL，例如System.String、System.Int32等。

- 内存管理 Memory Management
  <br>
  内存管理包括内存的释放和申请，例如.Net中创建一个对象会自动计算类型相关字段、类型对象指针和同步块索引占用的字节数，托管堆根据字节数分配内存，而不是像C/C++一样显式调用malloc申请一块内存区域。

- 线程管理 Thread Management
  <br>
  .Net中的线程和操作系统线程是对应的关系，.Net对操作系统线程进行了封装，并[附带一些托管环境下所需要的数据](http://www.cnblogs.com/JeffreyZhao/archive/2009/07/22/thread-pool-1-the-goal-and-the-clr-thread-pool.html)。
  另外，.Net会创建一个线程池来实现对线程的复用，以减少创建线程的开销。

- 内存自动回收 Garbage Collection
  <br>
  .Net实现了托管资源的自动内存管理（垃圾回收）。

- 安全性 Security
- 类型检查 Type Checker
- 异常管理 Exception Manager
- 除错管理 Debug Engine
- 中间码(MSIL)到机器码(Native)编译

- 类装载 Class Loader
  <br>
  Class Loader 是用来加载类型、程序集和模块等的。并且，Class Loader 是按需加载的，起到降低系统资源占用和启动加速的作用。
  JIT 在编译一个方法时会检查该方法引用的类型，Class Loader 就是在这个时候对程序集进行加载。
  这里其实还涉及到一个 Class Loader 对程序集定位的问题， Class Loader 定位程序集有两种方式，一种是根据文件路径定位，另一种是根据程序集名称，后者有些复杂，有时间再具体整理。
  
