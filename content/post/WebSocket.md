+++
date = '2026-06-24T16:30:25+08:00'
draft = false
title = 'WebSocket'

summary = 'Use WebSocket By C++'

+++

# Use WebSocket By C++

## 1.Overview

![img](https://i-blog.csdnimg.cn/direct/0feb66a800854958b32c97de069e07ba.png)

### 1.The Reason Why WebSocket Protocol Is Created

在WebSocket出现之前，一直使用的是http协议，但是这个协议有个很大的弊端，它是一个"拉"协议，必须由一方向另一方发送请求，进行拉取数据。因此http协议广泛应用于CS结构。但是在我们的日常的社交软件中，例如QQ,WeChat等，当好友给我们发送消息后，我们不可能拉获得数据，而是应该给我们主动推送过来。为了解决这个问题，WebSocket协议应运而生。

**WebSocket 解决了 HTTP 短连接 + 单向通信的核心痛点，实现了客户端与服务器的长连接双向实时通信，让服务器拥有了主动向客户端推送数据的能力**。

