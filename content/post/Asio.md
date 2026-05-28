+++
date = '2026-05-28T23:18:12+08:00'
draft = false
title = 'Asio'

pin = true

summary = 'Boost.Asio is a cross-platform C++ library for network and low-level I/O programming, providing a consistent asynchronous model based on modern C++ methods'

+++

# Overview

## 1.Basic Boost.Asio architecture

***Boost.Asio 可用于对 I/O 对象（如套接字）执行同步和异步操作。在使用 Boost.Asio 之前，了解 Boost.Asio 的各个部分、您的程序以及它们如何协同工作，将有助于建立一个概念性的认识。作为入门示例，让我们考虑在套接字上执行连接操作时会发生什么。我们将从检查同步操作开始.***

### 1.Synchronous Operation

![img](https://boost.ac.cn/doc/libs/latest/doc/html/boost_asio/sync_op.png)

**每个程序**将至少拥有一个**I/O 执行上下文**，例如 `boost::asio::io_context` 对象、`boost::asio::thread_pool` 对象或 `boost::asio::system_context`。这个**I/O 执行上下文**代表了**程序**与**操作系统** I/O 服务之间的连接。

```c++
boost::asio::io_context ioContext;
```

为了执行I/O操作，程序必须需要一个 I/O对象，也就是socket

```c++
boost::asio::ip::tcp::socket mySocket{ioContext};
```

当执行同步连接操作时，会发生以下事件序列：

1. **程序**通过调用**I/O 对象**来发起连接操作。

  ```c++
  mySocket.connect(serverEndpoint);
  ```

2. **I/O 对象**将请求转发给**I/O 执行上下文**。
3. **I/O 执行上下文**调用**操作系统**来执行连接操作。
4.  **操作系统**将操作结果返回给**I/O 执行上下文**。
5. **I/O 执行上下文**将操作导致的任何错误转换为 `boost::system::error_code` 类型的对象。`error_code` 可以与特定值进行比较，或者作为布尔值进行测试（其中 `false` 结果表示没有发生错误）。结果随后被转发回**I/O 对象**。
6. 如果操作失败，**I/O 对象**会抛出 `boost::system::system_error` 类型的异常。如果发起操作的代码改为这样编写：

```c++
boost::system::error_code ec;
mySocket.connect(serverEndpoint, ec);
```

那么 `error_code` 变量 `ec` 将被设置为操作结果，且不会抛出任何异常。

### 2.Asynchronous Operation

![img](https://boost.ac.cn/doc/libs/latest/doc/html/boost_asio/async_op1.png)

1. **程序**通过调用**I/O 对象**来发起连接操作

```c++
mySocket.async_connect(serverEndpoint, yourCompletionHandler);
```

其中 `yourCompletionHandler` 是一个函数或函数对象，其签名如下：

```c++
void yourCompletionHandler(const boost::system::error_code& ec);
```

所需的精确签名取决于正在执行的异步操作。参考文档指出了每种操作的适当形式。

2. **I/O 对象**将请求转发给**I/O 执行上下文**。
3. **I/O 执行上下文**向**操作系统**发出信号，要求其开始异步连接。

时间流逝。（在同步情况下，此等待将完全包含在连接操作的持续时间内。）

![img](https://boost.ac.cn/doc/libs/latest/doc/html/boost_asio/async_op2.png)

4. **操作系统**通过将结果放入队列来表示连接操作已完成，准备好被**I/O 执行上下文**提取。
5. 当使用 `ioContext` 作为**I/O 执行上下文**时，**程序**必须调用 `ioCcontext::run()`（或类似的 `ioCcontext` 成员函数）以便获取结果。调用 `ioContext::run()` 会在有未完成的异步操作时阻塞，因此您通常应在启动第一个异步操作后立即调用它。
6. 在 `ioContext::run()` 调用内部，**I/O 执行上下文**将操作结果出队，将其转换为 `error_code`，然后将其传递给**您的完成处理程序**。

# Tutorial

## 1.Basic skill

***本节的教程程序介绍使用 Asio 工具箱所需的基本概念。在深入网络编程的复杂世界之前，这些教程程序通过简单的异步计时器演示基本技能。***

### 1.Timer.1- Synchronize the use of timers

```c++
#include<boost/asio.hpp>
#include<iostream>
int main(int argc, const char* argv[]) {
	//1.创建上下文
	boost::asio::io_context io;
	//2.创建定时器
	boost::asio::steady_timer t{ io,boost::asio::chrono::seconds(5) };
	//3.进行等待
	t.wait();
	std::cout << "hello world" << std::endl;
}
```

### 2.Timer.1- Asynchronize the use of timers

```c++
#include<boost/asio.hpp>
#include<iostream>
//相当于一个回调函数作为完成的令牌
void print(const boost::system::error_code&) {
	std::cout << "hello world" << std::endl;
}
int main(int argc, const char* argv[]) {
	//创建上下文
	boost::asio::io_context io;
	//设置定时器
	boost::asio::steady_timer t{ io,boost::asio::chrono::seconds(5) };
	//注册回调函数
	t.async_wait(print);
	//调用run函数，确保所有的异步请求全部处理完成
	io.run();
}
```



