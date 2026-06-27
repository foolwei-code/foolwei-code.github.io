+++
date = '2026-06-18T12:16:11+08:00'
draft = false
title = 'SpdLog'

summary = 'spdlog is a high-performance C++ logging library'

+++

# spdlog is a high-performance C++ logging library

## 1.Install

```shell
$ git clone https://github.com/gabime/spdlog.git
$ cd spdlog && mkdir build && cd build
$ cmake ../
$ make&&make install
```

## 2.Usage samples

### 1.Console Printing

```c++
#include <spdlog/spdlog.h>
int main(int argc, const char *argv[]) {
  // 普通打印
  spdlog::info("Welcome to spdlog !");
  // 格式化打印
  // 1.打印字符串
  spdlog::info("Welcome to {}", "hello world");
  // 2.打印数字
  spdlog::error("spdlog error code is {}", 400);
  // 3. 打印占位符
  spdlog::warn("spdlog format char {:08d}",
               12); // 这里表示打印出12，但要足8位，如果不足，在前面补0
  // 4.格式话打印不同进制的数据
  spdlog::critical("Support for int:{0:d} hex:{0:x} oct:{0:o} bin:{0:b}", 12);
  // 5.打印浮点数数据
  spdlog::info("float args are {0:03.2f}", 1.23456);
  // 6.打印多个参数
  spdlog::info("string args are {0},{1}..", "too", "supported");
  spdlog::info("number args are {0},{1},{2}", 12, 13, 14);
}
```

![image-20260618131441153](https://cdn.jsdelivr.net/gh/foolwei-code/pictureBed@main/images/20260618131508413.png)

### 2.Print the log in the file

```c++
#include <iostream>
#include <spdlog/common.h>
#include <spdlog/sinks/basic_file_sink.h>
#include <spdlog/spdlog.h>
int main(int argc, const char *argv[]) {
  try {
    // 参数1 日志标识符, 参数2 日志文件名
    std::shared_ptr<spdlog::logger> mylogger =
        spdlog::basic_logger_mt("spdlog", "spdlog.log");
    // 设置日志格式. 参数含义: [日志标识符] [日期] [日志级别] [线程号] [数据]
    mylogger->set_pattern("[%n][%Y-%m-%d %H:%M:%S.%e] [%l] [%t]  %v");
    //设置打印的最低级别(trace<debug<info<warn<error<critical)
    mylogger->set_level(spdlog::level::debug);
    //每5s中刷新一次缓冲区
    spdlog::flush_every(std::chrono::seconds(5)); // 定期刷新日志缓冲区
    // 由于最低级别是debug,所以这条trace日志不会被打印
    mylogger->trace("Welcome to info spdlog!");
    mylogger->debug("Welcome to info spdlog!");
    mylogger->info("Welcome to info spdlog!");
    mylogger->warn("Welcome to info spdlog!");
    mylogger->error("Welcome to info spdlog!");
    mylogger->critical("Welcome to info spdlog!");

    // 刷新
    mylogger->flush_on(spdlog::level::debug);

  } catch (const spdlog::spdlog_ex &e) {
    std::cout << "Log initialization failed: " << e.what() << "\n";
  }
}
```

你可以在你的输出目录下找到spdlog.log

![image-20260618143945319](https://cdn.jsdelivr.net/gh/foolwei-code/pictureBed@main/images/20260618143948487.png)

* 输出内容

```shell
[spdlog][2026-06-18 13:43:33.525] [debug] [3811940]  Welcome to info spdlog!
[spdlog][2026-06-18 13:43:33.525] [info] [3811940]  Welcome to info spdlog!
[spdlog][2026-06-18 13:43:33.525] [warning] [3811940]  Welcome to info spdlog!
[spdlog][2026-06-18 13:43:33.525] [error] [3811940]  Welcome to info spdlog!
[spdlog][2026-06-18 13:43:33.525] [critical] [3811940]  Welcome to info spdlog!

```



### 3.Print in multiple files

```c++
#pragma once
#include <spdlog/spdlog.h>
void MyFunc() {
  std::shared_ptr<spdlog::logger> mylogger = spdlog::get("spdlog");

  mylogger->trace("Welcome to myFuncTest!");
  mylogger->debug("Welcome to myFuncTest!");
  mylogger->info("Welcome to myFuncTest!");
  mylogger->error("Welcome to myFuncTest!");
}
```

````c++
#include <iostream>
#include <spdlog/common.h>
#include <spdlog/sinks/base_sink.h>
#include <spdlog/sinks/basic_file_sink.h>
#include <spdlog/spdlog.h>
#include"MyFunc.hpp"
int main(int argc, const char *argv[]) {
  try {
    // 参数1 日志标识符, 参数2 日志文件名
    std::shared_ptr<spdlog::logger> mylogger =
        spdlog::basic_logger_mt("spdlog", "spdlog.txt");
    // 设置日志格式. 参数含义: [日志标识符] [日期] [日志级别] [线程号] [数据]
    mylogger->set_pattern("[%n][%Y-%m-%d %H:%M:%S.%e] [%l] [%t]  %v");
    mylogger->set_level(spdlog::level::debug);
    spdlog::flush_every(std::chrono::seconds(5)); // 定期刷新日志缓冲区

    mylogger->trace("Welcome to info spdlog!");
    mylogger->debug("Welcome to info spdlog!");
    mylogger->info("Welcome to info spdlog!");
    mylogger->warn("Welcome to info spdlog!");
    mylogger->error("Welcome to info spdlog!");
    mylogger->critical("Welcome to info spdlog!");
    MyFunc();
    // 刷新
    mylogger->flush_on(spdlog::level::debug);
  } catch (spdlog::spdlog_ex &e) {
    std::cerr << "Log initializtion failed" << e.what() << "\n";
  }
}
````

```
[spdlog][2026-06-18 14:56:06.842] [debug] [3819529]  Welcome to info spdlog!
[spdlog][2026-06-18 14:56:06.842] [info] [3819529]  Welcome to info spdlog!
[spdlog][2026-06-18 14:56:06.842] [warning] [3819529]  Welcome to info spdlog!
[spdlog][2026-06-18 14:56:06.842] [error] [3819529]  Welcome to info spdlog!
[spdlog][2026-06-18 14:56:06.842] [critical] [3819529]  Welcome to info spdlog!
[spdlog][2026-06-18 14:56:06.842] [debug] [3819529]  Welcome to myFuncTest!
[spdlog][2026-06-18 14:56:06.842] [info] [3819529]  Welcome to myFuncTest!
[spdlog][2026-06-18 14:56:06.842] [error] [3819529]  Welcome to myFuncTest!
```

