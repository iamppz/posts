前几天看到一位朋友在帖子里讲到Runtime就是一套API，感觉有些片面，在这里整理一下我的理解。

Runtime在.Net中叫CLR，在Java中叫JRE。

以CLR（公共语言运行时）举例，CLR是微软对CLI的实现，[CLR运行一种被称为CIL的语言的字节码](https://zh.wikipedia.org/wiki/%E9%80%9A%E7%94%A8%E8%AA%9E%E8%A8%80%E9%81%8B%E8%A1%8C%E5%BA%AB)。编译器将C#、F#等语言的代码编译成CIL，程序运行的时候CLR再通过JIT将CIL编译成机器码。

CLR提供以几个主要功能：
- 基类库支持 Base Class Library Support
- 内存管理 Memory Management
- 线程管理 Thread Management
- 内存自动回收 Garbage Collection
- 安全性 Security
- 类型检查 Type Checker
- 异常管理 Exception Manager
- 除错管理 Debug Engine
- 中间码(MSIL)到机器码(Native)编译
- 类装载 Class Loader
