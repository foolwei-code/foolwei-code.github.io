+++
date = '2026-05-02T14:48:57+08:00'
draft = false
title = 'MySQL'

pin = true

summary = 'MySQL Programming For C++'

+++

# MySQL Programming For C++

## 1.Preparation of the development environment

### 1.Install MySQL Server

* 确保已安装 MySQL Server，并创建测试数据库和用户

```mysql
CREATE DATABASE testdb;  
CREATE USER 'testuser'@'localhost' IDENTIFIED BY '123456';
GRANT ALL ON testdb.* TO 'testuser'@'localhost';
```

### 2.Install MySQL Client

```shell
# 对于Ubuntu系统
sudo apt update
sudo apt install libmysqlclient-dev

#对于CentOS系统
sudo yum install mysql-devel
```

## 2.The basic process of connecting to MySQL in C++

MySQL 官方提供的是 **C API**，C++ 可直接调用：

> 初始化 → 连接 → 执行 SQL → 处理结果 → 释放资源

## 3.Core API Description

|        **函数**        |     **作用**      |
| :--------------------: | :---------------: |
|     `mysql_init()`     | 初始化 MYSQL 对象 |
| `mysql_real_connect()` |    连接数据库     |
|    `mysql_query()`     |     执行 SQL      |
| `mysql_store_result()` |   获取查询结果    |
|  `mysql_fetch_row()`   |   读取一行数据    |
| `mysql_free_result()`  |    释放结果集     |
|    `mysql_close()`     |     关闭连接      |

## 4.Encapsulate a MySQL client

### 1.Head File

```c++
#pragma once
#include <mysql/mysql.h>
#include <string>
// 封装一个MySQL的客户端
class MySQLClient {
public:
  MySQLClient();
  ~MySQLClient();
  // 进行连接
  bool connect(const std::string &host, const std::string &user,
               const std::string &pass, const std::string &db,
               unsigned short port =
                   3306);
  // 判断是否连接成功
  bool isConnected() const;
  // 执行mysql的增删改语句
  bool execute(const std::string &sql) const;
  // 执行mysql中的查询语句
  MYSQL_RES *query(const std::string &sql) const;
  // 获取最后一次插入的数据的主键id
  int64_t lastInsertId() const;
  // 进行字符串的安全转义
  std::string escape(const std::string &str) const;

private:
  MYSQL *conn_;
};
```

### 2.Source File

```c++
#include "MySQLClient.hpp"
#include <cstring>
MySQLClient::MySQLClient() : conn_(nullptr) {}
// 释放连接资源
MySQLClient::~MySQLClient() {
  if (conn_)
    mysql_close(conn_);
}
// 进行连接
bool MySQLClient::connect(const std::string &host, const std::string &user,
                          const std::string &pass, const std::string &db,
                          unsigned short port) {
  // 设置连接资源
  conn_ = mysql_init(nullptr);
  if (!conn_)
    return false;
  // 设置字符集的编码方式为utif8mb4
  mysql_options(conn_, MYSQL_SET_CHARSET_NAME, "utf8mb4");
  // 与服务器建立连接
  return mysql_real_connect(conn_, host.c_str(), user.c_str(), pass.c_str(),
                            db.c_str(), port, nullptr, 0) != nullptr;
}
// 确认连接是否成功建立
bool MySQLClient::isConnected() const { return conn_ != nullptr; }
// 执行mysql的增删改语句
bool MySQLClient::execute(const std::string &sql) const {
  return mysql_query(conn_, sql.c_str()) == 0;
}
// 执行查询语句
MYSQL_RES *MySQLClient::query(const std::string &sql) const {
  if (mysql_query(conn_, sql.c_str()))
    return nullptr;
  return mysql_store_result(conn_);
}
// 获取最后一次插入的数据的主键id
int64_t MySQLClient::lastInsertId() const { return mysql_insert_id(conn_); }
// 进行字符串的安全转义
std::string MySQLClient::escape(const std::string &str) const {
  char *buf = new char[str.size() * 2 + 1];
  mysql_real_escape_string(conn_, buf, str.c_str(), str.size());
  std::string res(buf);
  delete[] buf;
  return res;
}
```

