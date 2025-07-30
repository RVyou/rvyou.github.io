---
title: Rust async/await
author: rvyou
date: 2022-04-09 16:23:10 +0800
categories: [Rust,异步并发,async/await,(1)]
tags: [Rust,异步并发,async/await]
---

最近看了很多 async/await 文章，做一下总结(最好理解版本)

### Future概念
 - 它代表一个尚未完成的异步计算。它不是计算结果本身，而是一个最终会产生结果（或错误）的操作句柄。
 - 基于 Poll 的模型： Rust 的 Future 采用轮询 (Polling) 模型。这意味着你需要反复 **询问** Future：**你完成了吗？** 直到它最终完成。这与 Stackful 协程那种可以任意暂停和恢复的方式有本质区别。
 - 三个阶段：
   - Poll (轮询) 阶段: 执行器 (Executor) 调用 Future 的 poll 方法，推动计算前进，直到遇到无法立即完成的点（如等待 I/O）。
   - Wait (等待) 阶段: 当 Future 无法继续前进时，它会安排一个**唤醒**机制（通过 Waker），通常是注册到反应器 (Reactor) 上，等待某个外部事件（如 I/O 就绪）。此时 Future 返回 Pending。
   - Wake (唤醒) 阶段: 当外部事件发生，Reactor 触发之前注册的 Wake。Wake 通知 Executor。
   - 再次 Poll: Executor 收到通知后，会再次调度并调用 Future 的 poll 方法，继续计算，直到最终返回 Ready(result)。

### 异步io：epoll/kqueue/IOCP

| **机制** | **操作系统**  | **核心原理**|
| ---- | --------- | ---------------------------------|
| epoll | Linux     | 基于红黑树+就绪链表，仅返回活跃的文件描述符（FD）。    |
| kqueue | BSD/macOS | 支持多种事件类型（文件、信号、定时器等），通过事件过滤器管理。 |
| IOCP | Windows   | 基于端口模型，与线程池结合实现异步I/O。        



### async fn 和.await 干了什么
这很重要，感觉能理解这里基本能理解全貌了

#### 1.async fn 定义 -> 返回 Future:
当定义一个 async fn my_func(...) -> T 时，编译器确实会将其签名（在内部）转换成一个返回实现了 Future<Output = T> 的类型的普通函数。这个返回的类型是一个匿名的、编译器生成的结构体。
#### 2.代码逻辑以 .await 为拆分点 -> 状态机的状态:
  - 编译器分析 async fn 的函数体，并将每个 .await 视为一个潜在的暂停点（yield point）。
  - 整个函数的执行流程被建模成一个状态机。这个状态机通常用一个 enum 来表示当前处于哪个状态。
  - 状态大致可以对应：
    - State0: 函数开始执行，直到第一个 .await。
    - State1: 第一个 .await 完成后，执行代码直到第二个 .await。
    - StateN: 第 N 个 .await 完成后，执行代码直到第 N+1 个 .await 或函数返回。
    - Terminated/Done: 函数执行完毕。
#### 3.每个状态存储下个 .await 可见的变量和 future:
  - 当函数在某个 .await 处暂停时，所有在暂停点之后的代码执行所必需的局部变量的值都必须被保存下来。
  - 这些变量会被移动（move）或借用（borrow）到编译器生成的那个状态机结构体的字段中。
  - 特别当执行到 let result = some_future.await; 时：
    - 如果 some_future.poll() 返回 Poll::Pending，那么 some_future 本身（或者对它的引用/Pin）也需要被存储在状态机结构体中，以便在下次 poll 时可以继续 poll 它。
    - 状态机的当前状态会被设置为“正在等待 some_future”。
#### 4.两个 .await 之间的代码 -> 状态机的转移逻辑:
  - 状态机结构体会实现 Future trait，核心是 poll 方法。
  - 当状态机的 poll 方法被调用时：
    - 它检查当前的状态 (enum 的变体)。
    - 根据当前状态，它执行相应的代码块（即两个 .await 之间的那段代码）。
    - 如果这段代码执行完成并遇到了下一个 .await some_future：
       - 它会去 poll 这个 some_future。
       - 如果 some_future 返回 Poll::Ready(value)，它会把 value 存起来（比如赋值给 result），然后立即继续执行 .await 后面的代码（可能进入下一个状态对应的逻辑，在同一次 poll 调用中）。
       - 如果 some_future 返回 Poll::Pending，状态机就会：
         - 保存好当前的局部变量状态。
         - 将自身的状态更新为“正在等待 some_future”。
         - 向调用者（Tokio 执行器）返回 Poll::Pending。
    - 如果代码执行到函数末尾并返回值 v，poll 方法就返回 Poll::Ready(v)。

### 总结
这样就好理解：
异步 io 会唤醒 Poll::Pending 让他们就绪，当遇到下一个.await 新一轮状态机。


### 参考

- [异步/等待](https://os.phil-opp.com/async-await/#the-async-await-pattern)

