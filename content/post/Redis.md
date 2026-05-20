+++
date = '2026-05-02T17:07:41+08:00'
draft = false
title = 'Redis'

pin = true

summary = 'Redis Programming For C++'

+++

# Redis Programming For C++

## 1.Preparation of the development environment

### 1.Install Redis

- 确保已安装 redis

```
redis --version
```



### 2.Clone hiredis

```shell
$ git clone https://github.com/redis/hiredis.git
$ cd hiredis/

$ mkdir build && cd build

$ cmake ..
$ make install
```

## 2.The basic process of connecting to Redis in C++

hiredis提供的是 **C API**，C++ 可直接调用：

> 初始化 → 连接 → 执行 redis → 处理结果 → 释放资源

## 3.Synchronous API

To consume the synchronous API, there are only a few function calls that need to be introduced:

```c++
redisContext *redisConnect(const char *ip, int port);
void *redisCommand(redisContext *c, const char *format, ...);
void freeReplyObject(void *reply);
```

### Connecting

The function `redisConnect` is used to create a so-called `redisContext`. The context is where Hiredis holds state for a connection. The `redisContext` struct has an integer `err` field that is non-zero when the connection is in an error state. The field `errstr` will contain a string with a description of the error. More information on errors can be found in the **Errors** section. After trying to connect to Redis using `redisConnect` you should check the `err` field to see if establishing the connection was successful:

```c++
redisContext *c = redisConnect("127.0.0.1", 6379);
  if (c == nullptr || c->err) {
    if (c) {
        std::printf("Error: %s\n", c->errstr);
        // handle error
    } else {
        std::printf("Can't allocate redis context\n");
    }
}
```

*Note: A `redisContext` is not thread-safe.*



### Sending commands

There are several ways to issue commands to Redis. The first that will be introduced is `redisCommand`. This function takes a format similar to printf. In the simplest form, it is used like this:

```c++
reply = redisCommand(context, "SET foo bar");
```

The specifier `%s` interpolates a string in the command, and uses `strlen` to determine the length of the string:

```c++
reply = redisCommand(context, "SET foo %s", value);
```



When you need to pass binary safe strings in a command, the `%b` specifier can be used. Together with a pointer to the string, it requires a `size_t` length argument of the string:

```c++
reply = redisCommand(context, "SET foo %b", value, (size_t) valuelen);
```



Internally, Hiredis splits the command in different arguments and will convert it to the protocol used to communicate with Redis. One or more spaces separates arguments, so you can use the specifiers anywhere in an argument:

```c++
reply = redisCommand(context, "SET key:%s %s", myid, value);
```



### Using replies

The return value of `redisCommand` holds a reply when the command was successfully executed. When an error occurs, the return value is `NULL` and the `err` field in the context will be set (see section on **Errors**). Once an error is returned the context cannot be reused and you should set up a new connection.

The standard replies that `redisCommand` are of the type `redisReply`. The `type` field in the `redisReply` should be used to test what kind of reply was received:

#### RESP2

- **`REDIS_REPLY_STATUS`**:
  - The command replied with a status reply. The status string can be accessed using `reply->str`. The length of this string can be accessed using `reply->len`.
- **`REDIS_REPLY_ERROR`**:
  - The command replied with an error. The error string can be accessed identical to `REDIS_REPLY_STATUS`.
- **`REDIS_REPLY_INTEGER`**:
  - The command replied with an integer. The integer value can be accessed using the `reply->integer` field of type `long long`.
- **`REDIS_REPLY_NIL`**:
  - The command replied with a **nil** object. There is no data to access.
- **`REDIS_REPLY_STRING`**:
  - A bulk (string) reply. The value of the reply can be accessed using `reply->str`. The length of this string can be accessed using `reply->len`.
- **`REDIS_REPLY_ARRAY`**:
  - A multi bulk reply. The number of elements in the multi bulk reply is stored in `reply->elements`. Every element in the multi bulk reply is a `redisReply` object as well and can be accessed via `reply->element[..index..]`. Redis may reply with nested arrays but this is fully supported.

#### RESP3

Hiredis also supports every new `RESP3` data type which are as follows. For more information about the protocol see the `RESP3` [specification.](https://github.com/antirez/RESP3/blob/master/spec.md)

- **`REDIS_REPLY_DOUBLE`**:
  - The command replied with a double-precision floating point number. The value is stored as a string in the `str` member, and can be converted with `strtod` or similar.
- **`REDIS_REPLY_BOOL`**:
  - A boolean true/false reply. The value is stored in the `integer` member and will be either `0` or `1`.
- **`REDIS_REPLY_MAP`**:
  - An array with the added invariant that there will always be an even number of elements. The MAP is functionally equivalent to `REDIS_REPLY_ARRAY` except for the previously mentioned invariant.
- **`REDIS_REPLY_SET`**:
  - An array response where each entry is unique. Like the MAP type, the data is identical to an array response except there are no duplicate values.
- **`REDIS_REPLY_PUSH`**:
  - An array that can be generated spontaneously by Redis. This array response will always contain at least two subelements. The first contains the type of `PUSH` message (e.g. `message`, or `invalidate`), and the second being a sub-array with the `PUSH` payload itself.
- **`REDIS_REPLY_ATTR`**:
  - An array structurally identical to a `MAP` but intended as meta-data about a reply. *As of Redis 6.0.6 this reply type is not used in Redis*
- **`REDIS_REPLY_BIGNUM`**:
  - A string representing an arbitrarily large signed or unsigned integer value. The number will be encoded as a string in the `str` member of `redisReply`.
- **`REDIS_REPLY_VERB`**:
  - A verbatim string, intended to be presented to the user without modification. The string payload is stored in the `str` member, and type data is stored in the `vtype` member (e.g. `txt` for raw text or `md` for markdown).

Replies should be freed using the `freeReplyObject()` function. Note that this function will take care of freeing sub-reply objects contained in arrays and nested arrays, so there is no need for the user to free the sub replies (it is actually harmful and will corrupt the memory).

**Important:** the current version of hiredis (1.0.0) frees replies when the asynchronous API is used. This means you should not call `freeReplyObject` when you use this API. The reply is cleaned up by hiredis *after* the callback returns. We may introduce a flag to make this configurable in future versions of the library.



### Cleaning up

To disconnect and free the context the following function can be used:

```
void redisFree(redisContext *c);
```

This function immediately closes the socket and then frees the allocations done in creating the context.



### Pipelining

To explain how Hiredis supports pipelining in a blocking connection, there needs to be understanding of the internal execution flow.

When any of the functions in the `redisCommand` family is called, Hiredis first formats the command according to the Redis protocol. The formatted command is then put in the output buffer of the context. This output buffer is dynamic, so it can hold any number of commands. After the command is put in the output buffer, `redisGetReply` is called. This function has the following two execution paths:

1. The input buffer is non-empty:
   - Try to parse a single reply from the input buffer and return it
   - If no reply could be parsed, continue at *2*
2. The input buffer is empty:
   - Write the **entire** output buffer to the socket
   - Read from the socket until a single reply could be parsed

The function `redisGetReply` is exported as part of the Hiredis API and can be used when a reply is expected on the socket. To pipeline commands, the only thing that needs to be done is filling up the output buffer. For this cause, two commands can be used that are identical to the `redisCommand` family, apart from not returning a reply:

```c++
void redisAppendCommand(redisContext *c, const char *format, ...);
void redisAppendCommandArgv(redisContext *c, int argc, const char **argv, const size_t *argvlen);
```



After calling either function one or more times, `redisGetReply` can be used to receive the subsequent replies. The return value for this function is either `REDIS_OK` or `REDIS_ERR`, where the latter means an error occurred while reading a reply. Just as with the other commands, the `err` field in the context can be used to find out what the cause of this error is.

The following examples shows a simple pipeline (resulting in only a single call to `write(2)` and a single call to `read(2)`):

```c++
redisReply *reply;
redisAppendCommand(context,"SET foo bar");
redisAppendCommand(context,"GET foo");
redisGetReply(context,(void**)&reply); // reply for SET
freeReplyObject(reply);
redisGetReply(context,(void**)&reply); // reply for GET
freeReplyObject(reply);
```



This API can also be used to implement a blocking subscriber:

```c++
reply = redisCommand(context,"SUBSCRIBE foo");
freeReplyObject(reply);
while(redisGetReply(context,(void *)&reply) == REDIS_OK) {
    // consume message
    freeReplyObject(reply);
}
```



### Errors

When a function call is not successful, depending on the function either `NULL` or `REDIS_ERR` is returned. The `err` field inside the context will be non-zero and set to one of the following constants:

- **`REDIS_ERR_IO`**: There was an I/O error while creating the connection, trying to write to the socket or read from the socket. If you included `errno.h` in your application, you can use the global `errno` variable to find out what is wrong.
- **`REDIS_ERR_EOF`**: The server closed the connection which resulted in an empty read.
- **`REDIS_ERR_PROTOCOL`**: There was an error while parsing the protocol.
- **`REDIS_ERR_OTHER`**: Any other error. Currently, it is only used when a specified hostname to connect to cannot be resolved.

In every case, the `errstr` field in the context will be set to hold a string representation of the error.

## 4.Encapsulate a Redis client

### Head File

```c++
#pragma once
#include <hiredis/hiredis.h>
#include <string>
#include<vector>
class RedisClient {
public:
  RedisClient();
  ~RedisClient();
  // 建立连接
  bool connect(const std::string &host = "127.0.0.1", int port = 6379);
  //输入密码
  void inputPassword(const std::string&password);
  // 确认连接是否建立
  bool isConnected() const;
  //通用命令
  // 1.查看所有的键
  std::vector<std::string> getAllKeys();
  // 2.查看键是否存在
  bool existsKey(const std::string &key);
  // 3.删除键
  bool del(const std::string &key);
  // 4.设置键的过期时间
  bool expire(const std::string &key, uint32_t seconds);
  // 5.查看键的存活时间
  int64_t existTime(const std::string &key);
  // 6.移除键的过期时间
  bool persist(const std::string &key);
  // 7.查看当前键的类型
  std::string keyType(const std::string &key);
  // 8.清空当前数据库
  void flushDB();
  // 9.清空所有数据库
  void flushAll();

  // String命令
  // 1.设置字符串
  bool stradd(const std::string &key, const std::string &value);
  // 2.设置有时间限制的字符串
  bool stretex(const std::string &key, uint32_t seconds,
             const std::string &value);
  // 3.获得字符串值
  std::string get(const std::string &key);
  // 4.批量获取字符串
  bool strmget(const std::vector<std::string> &keys);

  // Set命令
  bool sadd(const std::string &key, const std::vector<std::string> &members);
  bool sadd(const std::string &key, const std::string &member); // 单个成员重载
  bool srem(const std::string &key, const std::vector<std::string> &members);
  bool srem(const std::string &key, const std::string &member);
  std::vector<std::string> smembers(const std::string &key);
  bool sismember(const std::string &key, const std::string &member);
  size_t scard(const std::string &key);
  std::string spop(const std::string &key);        // 随机弹出一个
  std::string srandmember(const std::string &key); // 随机返回一个（不移除）
private:
  redisContext *context_;
};
```

### Source File

```c++
#include "RedisClient.hpp"
#include <hiredis/hiredis.h>
#include <hiredis/read.h>
#include <stdexcept>
RedisClient::RedisClient() : context_(nullptr) {}
// 释放连接
RedisClient::~RedisClient() {
  if (context_)
    redisFree(context_);
}
// 建立连接
bool RedisClient::connect(const std::string &host, int port) {
  context_ = redisConnect(host.c_str(), port);
  if (!context_ || context_->err)
    return false;
  return true;
}
// 输入密码
void RedisClient::inputPassword(const std::string &password) {
  auto reply = static_cast<redisReply *>(
      redisCommand(context_, "AUTH %s", password.c_str()));
  if (!reply) {
    throw std::runtime_error("AUTH command failed: no reply");
  }
  if (reply->type == REDIS_REPLY_ERROR) {
    std::string err = reply->str;
    freeReplyObject(reply);
    throw std::runtime_error("AUTH failed: " + err);
  }
  if (reply->type != REDIS_REPLY_STATUS) {
    freeReplyObject(reply);
    throw std::runtime_error("Unexpected reply type for AUTH");
  }
  freeReplyObject(reply);
}
// 确认连接是否建立
bool RedisClient::isConnected() const { return context_ && context_->fd != -1; }

// 通用命令
// 1.查看所有的键
std::vector<std::string> RedisClient::getAllKeys() {
  std::vector<std::string> keys;
  auto reply = static_cast<redisReply *>(redisCommand(context_, "KEYS *"));
  if (!reply)
    return keys;
  if (reply->type == REDIS_REPLY_ARRAY) {
    int32_t keyCount = reply->elements;
    for (int i = 9; i < keyCount; i++) {
      // 数组里每个元素都是 string 类型
      redisReply *ele = reply->element[i];
      keys.emplace_back(ele->str);
    }
  }
  freeReplyObject(reply);
  return keys;
}
// 2.查看键是否存在
bool RedisClient::existsKey(const std::string &key) {
  auto reply = static_cast<redisReply *>(
      redisCommand(context_, "EXISTS %s", key.c_str()));
  bool ok = reply && reply->integer > 0;
  freeReplyObject(reply);
  return ok;
}
// 3.删除键
bool RedisClient::del(const std::string &key) {
  auto reply =
      static_cast<redisReply *>(redisCommand(context_, "DEL %s", key.c_str()));
  bool ok = reply && reply->integer > 0;
  freeReplyObject(reply);
  return ok;
}
// 4.设置键的过期时间
bool RedisClient::expire(const std::string &key, uint32_t seconds) {
  auto reply = static_cast<redisReply *>(
      redisCommand(context_, "EXPIRE %s %d", key.c_str(), seconds));
  bool ok = reply && reply->integer > 0;
  freeReplyObject(reply);
  return ok;
}
// 5.查看键的存活时间
int64_t RedisClient::existTime(const std::string &key) {
  auto reply =
      static_cast<redisReply *>(redisCommand(context_, "TTL %s", key.c_str()));
  if (!reply || reply->type != REDIS_REPLY_INTEGER)
    return -3; //-3表示获取错误
  return reply->integer;
}
// 6.移除键的过期时间
bool RedisClient::persist(const std::string &key) {
  auto reply = static_cast<redisReply *>(
      redisCommand(context_, "PERSIST %s", key.c_str()));
  bool ok = reply && reply->integer > 0;
  freeReplyObject(reply);
  return ok;
}
// 7.查看当前键的类型
std::string RedisClient::keyType(const std::string &key) {
  auto reply =
      static_cast<redisReply *>(redisCommand(context_, "TYPE %s", key.c_str()));
  std::string type = reply->str;
  freeReplyObject(reply);
  return type;
}
// 8.清空当前数据库
void RedisClient::flushDB() {
  auto reply = static_cast<redisReply *>(redisCommand(context_, "FLUSHDB"));
  freeReplyObject(reply);
}
// 9.清空所有数据库
void RedisClient::flushAll() {
  auto reply = static_cast<redisReply *>(redisCommand(context_, "FLUSHALL"));
  freeReplyObject(reply);
}

/ String命令
// 1.设置字符串
bool RedisClient::stradd(const std::string &key, const std::string &value) {
  auto reply = static_cast<redisReply *>(
      redisCommand(context_, "SET %s %s", key.c_str(), value.c_str()));
  if (!reply)
    return false;
  bool ok =
      (reply->type == REDIS_REPLY_STATUS && std::string{reply->str} == "OK");
  freeReplyObject(reply);
  return ok;
}
// 2.设置有时间限制的字符串
bool RedisClient::stretex(const std::string &key, uint32_t seconds,
                          const std::string &value) {
  auto reply = static_cast<redisReply *>(redisCommand(
      context_, "SETEX %s %u %s", key.c_str(), seconds, value.c_str()));
  if (!reply)
    return false;

  bool ok =
      (reply->type == REDIS_REPLY_STATUS && std::string(reply->str) == "OK");
  freeReplyObject(reply);
  return ok;
}
// 3.获得字符串值
std::string RedisClient::get(const std::string &key) {
  auto reply =
      static_cast<redisReply *>(redisCommand(context_, "GET %s", key.c_str()));
  if (!reply)
    return "";

  std::string value;
  if (reply->type == REDIS_REPLY_STRING) {
    value.assign(reply->str, reply->len);
  }
  freeReplyObject(reply);
  return value;
}
// 4.批量获取字符串
std::vector<std::string>
RedisClient::strmget(const std::vector<std::string> &keys) {
  std::vector<std::string> result;
  if (keys.empty()) {
    return result; // 空输入直接返回空vector
  }

  // 构造 "MGET key1 key2 ..." 命令字符串
  std::string cmd = "MGET";
  for (const auto &key : keys) {
    cmd += " " + key;
  }

  // 执行命令
  redisReply *reply =
      static_cast<redisReply *>(redisCommand(context_, cmd.c_str()));
  if (!reply) {
    // 命令执行失败，返回空vector（或可记录日志）
    return result;
  }

  // 检查返回类型是否为数组
  if (reply->type == REDIS_REPLY_ARRAY) {
    result.reserve(reply->elements);
    for (size_t i = 0; i < reply->elements; ++i) {
      redisReply *ele = reply->element[i];
      if (ele && ele->type == REDIS_REPLY_STRING) {
        // 存在且是字符串，按长度拷贝（二进制安全）
        result.emplace_back(ele->str, ele->len);
      } else {
        // key 不存在或其他类型（如 nil），返回空字符串
        result.emplace_back("");
      }
    }
  }
  // 如果返回类型不是数组（例如错误），result 保持为空

  freeReplyObject(reply);
  return result;
}

// Set命令

bool RedisClient::sadd(const std::string &key,
                       const std::vector<std::string> &members) {
  if (members.empty())
    return false;

  std::string cmd = "SADD " + key;
  for (const auto &m : members) {
    cmd += " " + m;
  }

  auto reply = static_cast<redisReply *>(redisCommand(context_, cmd.c_str()));
  if (!reply || reply->type != REDIS_REPLY_INTEGER) {
    if (reply)
      freeReplyObject(reply);
    return false;
  }
  bool ok = (reply->integer > 0);
  freeReplyObject(reply);
  return ok;
}

bool RedisClient::sadd(const std::string &key, const std::string &member) {
  return sadd(key, std::vector<std::string>{member});
}

bool RedisClient::srem(const std::string &key,
                       const std::vector<std::string> &members) {
  if (members.empty())
    return false;

  std::string cmd = "SREM " + key;
  for (const auto &m : members) {
    cmd += " " + m;
  }

  auto reply = static_cast<redisReply *>(redisCommand(context_, cmd.c_str()));
  if (!reply || reply->type != REDIS_REPLY_INTEGER) {
    if (reply)
      freeReplyObject(reply);
    return false;
  }
  bool ok = (reply->integer > 0);
  freeReplyObject(reply);
  return ok;
}

bool RedisClient::srem(const std::string &key, const std::string &member) {
  return srem(key, std::vector<std::string>{member});
}

std::vector<std::string> RedisClient::smembers(const std::string &key) {
  std::vector<std::string> result;
  auto reply = static_cast<redisReply *>(
      redisCommand(context_, "SMEMBERS %s", key.c_str()));
  if (!reply || reply->type != REDIS_REPLY_ARRAY) {
    if (reply)
      freeReplyObject(reply);
    return result;
  }

  result.reserve(reply->elements);
  for (size_t i = 0; i < reply->elements; ++i) {
    redisReply *ele = reply->element[i];
    if (ele && ele->type == REDIS_REPLY_STRING) {
      result.emplace_back(ele->str, ele->len);
    }
  }
  freeReplyObject(reply);
  return result;
}

bool RedisClient::sismember(const std::string &key, const std::string &member) {
  auto reply = static_cast<redisReply *>(
      redisCommand(context_, "SISMEMBER %s %s", key.c_str(), member.c_str()));
  if (!reply || reply->type != REDIS_REPLY_INTEGER) {
    if (reply)
      freeReplyObject(reply);
    return false;
  }
  bool exists = (reply->integer == 1);
  freeReplyObject(reply);
  return exists;
}

size_t RedisClient::scard(const std::string &key) {
  auto reply = static_cast<redisReply *>(
      redisCommand(context_, "SCARD %s", key.c_str()));
  if (!reply || reply->type != REDIS_REPLY_INTEGER) {
    if (reply)
      freeReplyObject(reply);
    return 0;
  }
  size_t count = static_cast<size_t>(reply->integer);
  freeReplyObject(reply);
  return count;
}

std::string RedisClient::spop(const std::string &key) {
  auto reply =
      static_cast<redisReply *>(redisCommand(context_, "SPOP %s", key.c_str()));
  if (!reply || reply->type != REDIS_REPLY_STRING) {
    if (reply)
      freeReplyObject(reply);
    return "";
  }
  std::string result(reply->str, reply->len);
  freeReplyObject(reply);
  return result;
}

std::string RedisClient::srandmember(const std::string &key) {
  auto reply = static_cast<redisReply *>(
      redisCommand(context_, "SRANDMEMBER %s", key.c_str()));
  if (!reply || reply->type != REDIS_REPLY_STRING) {
    if (reply)
      freeReplyObject(reply);
    return "";
  }
  std::string result(reply->str, reply->len);
  freeReplyObject(reply);
  return result;
}
```







