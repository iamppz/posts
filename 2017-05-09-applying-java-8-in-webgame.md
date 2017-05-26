---
layout: post
tags: java
author: zhangshaolei
title: "Java 8 新增特性简介与应用"
---

## Java 8 新增特性简介与应用

### Lambda

Lambda 表达式是一种匿名函数(对 Java 而言这并不完全正确，但现在姑且这么认为)，简单地说，它是没有声明的方法，也即没有访问修饰符、返回值声明和名字。

C# 从 3.0 开始支持 Lambda 表达式， JavaScript 则从 ES2015 开始引入这个特性。

那么 Lambda 可以带来什么，有什么意义？举个例子，以下是两个列表：

``` Java
List<Task> allTasks = taskBll.findAll();  // 所有的任务
List<UserTask> userTasks = userTaskBll.find(userId);  // 用户已完成的任务
```

在任务中心页面，我们想显示一个任务列表，显示当前登录的用户每个任务完成与否。

遍历实现：

``` Java
List<UserTaskCompletionInfo> list = new ArrayList<>();
for (Task task : allTasks) {
    UserTaskCompletionInfo userTaskCompletionInfo = new UserTaskCompletionInfo();
    userTaskCompletionInfo.setTaskId(task.getCode());
    // 略...
    for (UserTask userTask : userTasks) {
        if (userTask.getTask() == task.getCode()) {
            userTaskCompletionInfo.setComplete(true);
            break;
        }
    }
    list.add(task);
}
```

Lambda 实现：

``` Java
List<UserTaskCompletionInfo> list = allTasks.stream().map(t -> new UserTaskCompletionInfo() {{
    setTaskId(t.getCode());
    // 略...
    setComplete(userTasks.stream().anyMatch(ut -> ut.getTask() == t.getCode()));
}}).collect(Collectors.toList());
```

- `allTasks.stream().map()` 方法接收的参数类型为 `Function<? super T, ? extends R>` ，在这里我们直接定义了一个匿名函数 `t => new UserTaskCompletionInfo() {{ ... }}` ，并将其作为参数传递给 `map()` 方法，使程序的逻辑更紧凑。

- `t -> new UserTaskCompletionInfo() {{ ... }}` 其实是 `(t) -> { return new UserTaskCompletionInfo(); }` 的变体， `(t)` 是这个匿名函数的参数声明， `map()` 函数每遍历到一个对象，就调用匿名函数，将对象作为参数传递进去。在语句 `setComplete(userTasks.stream().anyMatch(ut -> ut.getTask() == t.getCode()));` 中，函数直接访问到了外部的 `userTasks` 变量，这是一个典型的闭包场景，从函数式编程的角度来说，支持闭包也是很重要的。闭包是指将当前作用域中的变量通过值或者引用的方式封装到 Lambda 表达式当中，成为表达式的一部分，它使你的 Lambda 表达式从一个普通的函数变成了一个带隐藏参数（`allTasks`）的函数。

- 关于函数式编程的流行还有以下（但不限于）几个观点：

  1. 摩尔定律不适用了 -> 堆CPU -> 并行编程；

  2. 接近自然语言 -> 易于理解；

  3. 代码简洁 -> 开发快速；

  4. OO（面向对象）本来就是异端。

### 方法引用

方法引用的语法有些类似 PHP 的范围解析符，例如 ExchangeService 的 buildPrizeInfo 方法，可以写成 `ExchangeService::buildPrizeInfo`。不过 Java 的方法引用相对 JavaScript 这种将函数设计为一等公民的语言来说使用范围来说是要小很多的，在 JavaScript 你可以这么写：

```JavaScript
function foo() {
}

function bar(func) {
	func()
}

bar(foo);
```

但是 Java 只能在应用 Lambda 表达式的场景中替代 Lambda 表达式使用。

当我们使用 Lambda 表达式，例如 `.stream().map()` ，表达式内容可能只调用一个方法，一般是这样写的：

```Java
List<Prize> prizes = prizeBll.get(type);

List<PrizeInfo> prizeInfoList = prizes.stream().map(prize -> buildPrizeInfo(prize))
    .collect(Collectors.toList());
```

使用方法引用：

```Java
List<Prize> prizes = prizeBll.get(type);

List<PrizeInfo> prizeInfoList = prizes.stream().map(ExchangeService::buildPrizeInfo)
    .collect(Collectors.toList());
```

方法引用有四种调用方式：

- 引用静态方法（接受一个参数）

```Java
String::valueOf
```

- 引用特定对象的实例方法

```Java
this::buildPrizeInfo
```

- 引用特定类型的任意对象的实例方法（不接收参数）

```Java
String::length
```

- 引用构造函数

```Java
PrizeInfo::new
```

### Stream API

在 ES2015 诞生之前， JavaScript 通过一些第三方库（如lodash、 underscore）来扩展数组操作。

``` JavaScript
// lodash
var names = _.chain(users).map(user -> user.user).join(" , ").value();
```

C# 则通过静态类型 System.Linq.Enumerable 为实现了 IEnumerable 接口的类型扩展了一系列的数组操作方法。

``` CSharp
var names = string.Join(",", users.Select(user => user.user).ToList());
```

不管是 lodash 的 map() 、 join() ，还是 C# 的 Select()，都是通过代码封装来简化程序逻辑，但是又并不仅仅是这些。

回过头来看 lodash 与 underscore 之争，下面的表格是 lodash 与 underscore 的性能（每秒事务数）比较，数据通过 http://danieltao.com/lazy.js/comparisons.html 生成，摘取三个较常用的函数：

function | lodash | underscore
----|----|----
map | 4,100,555.63 | 1,933,753.04
filter | 5,069,086.20 | 1,692,445.98
union | 302,326.58 | 234,624.13

抛开程序逻辑等等细节不谈，导致 undersocre 与 lodash 之间巨大差距的一个重要的原因是 lodash 使用了延迟计算，在语句中 `_.chain(users).map(user -> user.user).join(" , ").value();` 中，无论是执行到 `.map()` 还是 `.join()` ，程序并不是每一步都对数组进行求值，而是在 `.value()` 这一步合并链式 iteratee 降低迭代次数来提升性能；或是不调用 `.value()` ，而将查询缓存在变量中，等到真正调用再 `.value()` 进行求值，也可以起到节省内存的效果。

C# 的 `ToList()` 也是这个道理。

至于 Java 的 Stream 的工作原理，可能是这样：

``` Java
public Stream filter(Predicate p) {
    this.filter = p; // 保存 predicate
    return this; // 返回 Stream 对象，而不是 predicate 的计算结果
}
```

缓存每一次链式调用的 predicate （or 推导 or Lambda 表达式） ，通过 `.collect(Collectors.toList())` 来合并 predicate 并立即求值。

已上。因为我对 Java 并不是太了解，所以只能借助 JavaScript 和 C# 来说明 Stream API 的大致工作原理以及优点……

（未完）

### 参考资料

> https://zh.wikipedia.org/wiki/%CE%9B%E6%BC%94%E7%AE%97

> http://danieltao.com/lazy.js/comparisons.html

> http://www.jianshu.com/p/5b800057f2d8
