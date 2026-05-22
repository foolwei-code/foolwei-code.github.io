+++
date = '2026-05-22T14:38:17+08:00'
draft = false
title = 'ODB'

pin = true

summary = 'ODB is  cross-database object-relational mapping (ORM) system for C++'

+++

# ODB is  cross-database object-relational mapping (ORM) system for C++

## 1.Install ODB In Linux

***因为我的Linux系统是Ubuntu，所以我就以Ubuntu 24.04为例，来说明下载***

### 1.Open The Official Website Of ODB([ODB - C++ Object-Relational Mapping (ORM)](https://codesynthesis.com/products/odb/))

![img](https://cdn.jsdelivr.net/gh/foolwei-code/pictureBed@main/images/20260522144703543.png)

### 2.Choose the appropriate software package based on your Linux version

![image-20260522145057270](https://cdn.jsdelivr.net/gh/foolwei-code/pictureBed@main/images/20260522145110593.png)

### 3.Install Software Package

![image-20260522145309880](https://cdn.jsdelivr.net/gh/foolwei-code/pictureBed@main/images/20260522145313019.png)

因为我是用的是ODB+MySQL,所以下载这5个软件包

* 用这5个包进行安装

```shell
sudo apt install -y \
./odb_2.5.0-0~ubuntu24.04_amd64.deb \
./libodb_2.5.0-0~ubuntu24.04_amd64.deb \
./libodb-dev_2.5.0-0~ubuntu24.04_amd64.deb \
./libodb-mysql_2.5.0-0~ubuntu24.04_amd64.deb \
./libodb-mysql-dev_2.5.0-0~ubuntu24.04_amd64.deb
```





## 2.Use ODB

### I Will Provide A Piece Of CRUD Code To Briefly Describe The Use Of ODB

```c++
#pragma once
#include <odb/core.hxx>
#include <odb/forward.hxx>
#include <string>
#pragma db object table("student")
/*Define Student Entity Class */
class Student {
public:
  Student() = default;
  Student(std::string name, unsigned int age, std::string stuNo)
      : name_(std::move(name)), age_(age), stu_no_(std::move(stuNo)) {}

  // Member Access Interface
  std::string name() const { return name_; }
  unsigned int age() const { return age_; }
  std::string stuNo() const { return stu_no_; }

  void setName(const std::string &name) { name_ = name; }
  void setAge(int age) { age_ = age; }

private:
  /*Define Field*/
#pragma db id auto
  unsigned long id_;

#pragma db type("VARCHAR(255)")
  std::string name_;

  unsigned int age_;

#pragma db unique
#pragma db type("VARCHAR(255)")
  std::string stu_no_;

  /*provide a friend*/
  friend class odb::access;
};
```

```shell
odb --std c++20 -d mysql --generate-query --generate-schema student.hxx
```



```c++
#include "student-odb.hxx"
#include "student.hpp"
#include <exception>
#include <iostream>
#include <memory>
#include <odb/database.hxx>
#include <odb/mysql/database.hxx>
#include <odb/query.hxx>
#include <odb/result.hxx>
#include <odb/transaction.hxx>
#include <string>
#include <vector>
int main(int argc, const char *argv[]) {
  try {
    std::string user{"root"};
    std::string dbName{"ODB"};
    std::string password{"123456"};
    std::string host{"127.0.0.1"};
    int unsigned port{3306};
    std::unique_ptr<odb::mysql::database> db{
        new odb::mysql::database{user, password, dbName, host, port}};
    // 1.insert data into databse
    {
      odb::transaction trans{db->begin()};
      std::vector<Student> stuList{{"zhangsan", 12, "202601"},
                                   {"lisi", 13, "202602"},
                                   {"wangwu", 14, "202603"}};
      for (auto &p : stuList) {
        db->persist(p);
      }
      trans.commit();
      std::cout << "insert success" << std::endl;
    }

    // 2.fuzzy query data
    {
      odb::transaction trans{db->begin()};
      auto q{odb::query<Student>::name.like("zhang%")};
      odb::result<Student> res{db->query<Student>(q)};
      for (const auto &p : res) {
        std::cout << p.age() << " " << p.name() << " " << p.stuNo()
                  << std::endl;
      }
      trans.commit();
    }
    // 3.conditional combined query
    {
      odb::transaction trans{db->begin()};
      auto q{odb::query<Student>::age > 12 && odb::query<Student>::age < 14};
      odb::result<Student> res{db->query<Student>(q)};
      for (const auto &p : res) {
        std::cout << p.age() << " " << p.name() << " " << p.stuNo()
                  << std::endl;
      }
      trans.commit();
    }

    // 4.paging query
    {
      odb::transaction trans(db->begin());
      std::string sql = "ORDER BY id LIMIT 0,2";
      auto res = db->query<Student>(sql);
      for (auto &s : res) {
        std::cout << s.age() << " " << s.name()<<" "<<s.stuNo() << std::endl;
      }
      trans.commit();
    }

    // 5.delete data
    {
      odb::transaction trans{db->begin()};
      std::string sql{"WHERE name = 'lisi'"};
      db->erase_query<Student>(sql);
      trans.commit();
    }

    // 6.update data
    {
      odb::transaction trans{db->begin()};
      auto p{db->load<Student>(3)};
      p->setAge(29);
      db->update(p);
      trans.commit();
    }
  } catch (const std::exception &e) {
    std::cerr << e.what() << std::endl;
  }
}
```



```shell
cmake_minimum_required(VERSION 3.20)
project(DOB)
set(CMAKE_CXX_STANDARD 20)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_SOURCE_DIR}/bin)
add_executable(main main.cpp student-odb.cxx)
target_link_libraries(main odb odb-mysql)
```

