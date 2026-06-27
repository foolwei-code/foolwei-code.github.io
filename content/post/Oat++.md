+++
date = '2026-05-04T20:32:32+08:00'
draft = false
title = 'Oat++'

pin = true

summary = 'Introduce Oat++  An Open Source C++ Web Framework'

+++

#  Oat++ Framwork

## 1.Install Oat++

### 1.Linux

```shell
$ git --branch 1.3.0 clone https://github.com/oatpp/oatpp.git #因为直接clone会克隆最新版本1.4.0，而官方文档还是1.3.0，所以还是下载1.3.0
$ cd oatpp/

$ mkdir build && cd build

$ cmake ..
$ make install
```

### 2.Windows

```shell
$ git --branch clone https://github.com/oatpp/oatpp.git
$ cd oatpp\
$ MD build
$ cd build\

$ cmake ..
$ cmake --build . --target INSTALL
```

## 2.Step By Step Guide

这个分步指南将帮助您开始使用oatpp框架进行开发。完成后，您将拥有一个具有基本端点的结构良好且可扩展的web服务。

#### 1.A Simple Project

***为了获得基本的组件概述，让我们先看一下最简单的oatpp服务器应用程序。***

* 总体架构

```apl
客户端请求
    ↓
ConnectionProvider（监听 8000，接收 TCP 连接）
    ↓
ConnectionHandler（解析成 HTTP 请求）
    ↓
Router（根据 URL 找到对应 API）
    ↓
你的 Controller 业务代码
    ↓
返回响应 → 发给客户端
```

```apl
客户端 Browser / Postman /Apifox
        │
        ▼
┌─────────────────────────────┐
│ ConnectionProvider          │
│ 职责：监听IP+端口、接收原始TCP连接 │
└───────────┬─────────────────┘
            │ 投递TCP原始连接
            ▼
┌─────────────────────────────┐
│ HttpConnectionHandler       │
│ 职责：TCP 拆包/封包、解析HTTP协议 │
│ 把原始字节流封装成HTTP请求对象    │
└───────────┬─────────────────┘
            │ 转发解析好的HTTP请求
            ▼
┌─────────────────────────────┐
│ HttpRouter 路由调度器        │
│ 职责：匹配URL、分发到对应Controller │
└───────────┬─────────────────┘
            │
            ▼
┌─────────────────────────────┐
│ Controller 业务接口层       │
│ 你的接口业务逻辑：处理请求、构造响应 │
└─────────────────────────────┘

────────────────────────────────────────────
【顶层统筹】
       Server
把 ConnectionProvider + HttpConnectionHandler
绑定在一起，启动事件循环、持续运行服务
```



