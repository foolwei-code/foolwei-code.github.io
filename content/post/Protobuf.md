+++
date = '2026-05-02T21:52:45+08:00'
draft = false
title = 'Protobuf'

pin = true

summary = 'using Protobuf to Serialized data'

+++



# using Protobuf to Serialized data

## 1.overview

Protocol Buffers 是一种与语言无关、与平台无关的可扩展机制，用于序列化结构化数据。

它类似于 JSON，但更小更快，并且生成原生语言绑定。你只需定义一次数据的结构方式，然后就可以使用专门生成的源代码，轻松地将结构化数据写入和读取到各种数据流中，并使用各种语言。

Protocol Buffers 是定义语言（在 `.proto` 文件中创建）、proto 编译器生成的用于与数据交互的代码、特定于语言的运行时库、写入文件（或通过网络连接发送）的数据序列化格式以及序列化数据的组合。

## 2. install source



