+++
date = '2026-06-04T10:06:44+08:00'
draft = false
title = 'Python'

summary  = 'In this blog ,I will introduce the grammar of Python'

+++

# The Grammar Of Python

## 1.Python Basics: Variables and Logical Control

1. 单行注释和多行注释

单行注释:在Python中单行注释是使用#，多行注释是使用'''   '''或者是""" """

```python
#单行注释


'''
多行注释
'''
```

2. 变量的定义和使用

Python官方推荐变量命名使用下划线式命名法

* 变量数据类型

| 类型  |       类型说明        |                  举例                  |
| :---: | :-------------------: | :------------------------------------: |
|  int  |         整形          |                age = 20                |
| float |         浮点          |              price = 12.3              |
|  str  |        字符串         |           name = "zhangsan"            |
| bool  |         布尔          |              flag = True               |
| list  |  列表(有序可变集合)   |            array = [1,2,3]             |
| tuple | 元组 (有序不可变集合) |         info = ("zhangsan",12)         |
| dict  |   字典(键值对集合)    | student = {"name":"zhangsan","age":18} |

```Python
a = 10
print(type(a))
price = 12.3
print(type(price))
name = "zhangsan"
print(type(name))
flag = True
print(type(flag))
array = [1,2,3]
print(type(array))
info = ("name",12)
student = {"name":"zhangsan","age":12}
print(type(info))
print(type(student))
```

3. 输入和输出

***格式化输入和输出***

```python
name = "zhangsan"
age = 12
#基础用法
#输出函数
print(f"姓名:{name},年龄:{age}")
#精度控制
number = 12.345
print(f"浮点数{number:.2f}")
#数字补0
print(f"{age:06d}")

#输入函数
name2 = input("请输入你的姓名:")
age2 = input("请输入你的年龄:")
print(name2)
print(age2)
```

4. 数据类型转换

| **函数** | **功能**         | **示例**           |
| -------- | ---------------- | ------------------ |
| int(x)   | 转换为整数       | int("100") → 100   |
| float(x) | 转换为浮点数     | float("9.9") → 9.9 |
| str(x)   | 转换为字符串     | str(18) → "18"     |
| eval(x)  | 执行字符串表达式 | eval("1+2") → 3    |

5. 逻辑运算符 

| **运算符** | **描述**           | **示例**                  |
| ---------- | ------------------ | ------------------------- |
| and        | 逻辑与（都真才真） | (10>5) and (20>10) → True |
| or         | 逻辑或（有真就真） | (10>5) or (20<10) → True  |
| not        | 逻辑非（取反）     | not (10>5) → False        |

6. if条件判断

```python
# 语法
if 条件表达式:
    满足条件时执行的代码块（缩进）

# 示例：判断是否成年
age = 19
if age >= 18:
    print("你已成年，可独立出行")
```

```python
height = input("身高:")
weight = input("体重:")
BMI = weight / height
if BMI < 18.5:
    print("偏瘦")
elif BMI >= 18.5 and BMI < 23.9:
    print("正常")
elif BMI >= 23.9 and BMI < 27.9:
    print("偏重")
else:
    print("肥胖")

```

7. for while循环

```python
# 演示 while循环
"""
while 判断条件:
    # 当条件满足 需要执行的内容

说明:
    while: 实现循环操作的关键字
    条件判断: 是否进入循环的依据
        当条件成立, 就执行循环里面的内容, 条件一直成立 就一直循环执行内部的代码 直到条件不成立
"""

# 需求: 请循环 打印 5次  Python真有趣
# 1- 定义计数器
num = 1

# 2- 编写循环操作
while num <= 5:
    # 2.1 执行循环的主体代码
    print("Python真有趣")

    # 2.2 更新计数器 +1
    # num = num + 1
    num += 1
```

* break关键字
* continue关键字

```Python
# while循环
num = 0
while num < 5:
    print("你好")
    num += 1
sum = 0
i = 1
while i <= 50:
    sum += i
    i += 1
print(sum)
sum = 0
i = 0
while i <= 50:
    if i % 2 != 0:
        sum += i
    i += 1
print(sum)

i = 0
while i <= 10:
    if i == 3:
        break
    print(f"这是第{i}个数")
    i += 1

i = 0
while i <= 5:
    if i == 3:
        i += 1
        continue
    print(f"这是第{i}个数")
    i += 1
print("结束")

```

```Python
# 演示: for基本使用
"""
    语法格式:  
        for 临时变量 in 序列:
            循环执行的代码块
        
        注意:
            临时变量的值 其实 就是 序列中每一个元素，循环次数等于序列的长度
"""
# 需求: 遍历一个字符串 :  python 
for char in "python":
    print(char)
```

```python
i = 0
for num in range(0,4):
    print(num)
```

## 2. Basical DataStructure

### 1.List

```
列表序列名称 = [列表中的元素1, 列表中的元素2, 列表中的元素3, ...]

注意: 列表是可变的
```

```Python
# List列表
name_list = ["张三", "lisi", "wangwu"]
print(type(name_list))
print(name_list)
# 与其他语言比如c/c++/java中的数组有所区别的是list可以存储不同的类型
student = ["zhangsan", 12]
print(student)

# 打印列表中的某一位数
print(student[0])

#遍历列表
for elem in student:
    print(elem)
```

☆查操作的相关方法：

| **编号** | **函数** | **作用**                                                     |
| -------- | -------- | ------------------------------------------------------------ |
| 1        | index()  | 指定数据所在位置的下标                                       |
| 2        | count()  | 统计指定数据在当前列表中出现的次数                           |
| 3        | in       | 判断指定数据在某个列表序列，如果在返回True，否则返回False    |
| 4        | not in   | 判断指定数据不在某个列表序列，如果不在返回True，否则返回False |

☆增操作

| **编号** | **函数** | **作用**                                                     |
| -------- | -------- | ------------------------------------------------------------ |
| 1        | append() | 增加指定数据到列表中                                         |
| 2        | extend() | 列表结尾追加数据，如果数据是一个序列，则将这个序列的数据逐一添加到列表 |
| 3        | insert() | 指定位置新增数据                                             |

☆ 删操作

| **编号** | **函数**       | **作用**                                         |
| -------- | -------------- | ------------------------------------------------ |
| 1        | del 列表[索引] | 删除列表中的某个元素                             |
| 2        | pop()          | 删除指定下标的数据(默认为最后一个)，并返回该数据 |
| 3        | remove()       | 移除列表中某个数据的第一个匹配项。               |

☆ 改操作

| **编号** | **函数**                | **作用**               |
| -------- | ----------------------- | ---------------------- |
| 1        | 列表[索引] = 修改后的值 | 修改列表中的某个元素   |
| 2        | reverse()               | 将数据序列进行倒叙排列 |
| 3        | sort()                  | 对列表序列进行排序     |

###  2.Tuple

元组特点：定义元组使用小括号，且使用逗号隔开各个数据，数据可以是不同的数据类型。

注意：如果定义的元组只有一个数据，那么这个数据后面也要添加逗号，否则数据类型为唯一的这个数据的数据类型。

由于元组中的数据不允许直接修改，所以其操作方法大部分为查询方法。

| **编号** | **函数**   | **作用**                                                     |
| -------- | ---------- | ------------------------------------------------------------ |
| 1        | 元组[索引] | 根据索引下标查找元素                                         |
| 2        | index()    | 查找某个数据，如果数据存在返回对应的下标，否则报错，语法和列表、字符串的index方法相同 |
| 3        | count()    | 统计某个数据在当前元组出现的次数                             |
| 4        | len()      | 统计元组中数据的个数                                         |

```Python
#元组
num_tuple = (1,2,3,4)
for num in num_tuple:
    print(num)
print(num_tuple.index(3))
print(num_tuple.count(3))

print(len(num_tuple))
```

* 元组的拆包

```python
num_tuple = (1,2,3)
a,b,c = num_tuple
print(a,b,c)
```



### 3.Set

一个无序并且不重复的序列

| 编号 |    函数    |     作用     |
| :--: | :--------: | :----------: |
|  1   |    add     |     添加     |
|  2   |   remove   |     删除     |
|  3   | in /not in | 判断在与不在 |

```Python
#集合
num_set = {1,2,3,4,5,6,7,8,9}
num_set.add(10)
num_set.remove(10)
flag = 10 in num_set
print(flag)
```

### 4.Dict

```Python
集合
num_set = {1,2,3,4,5,6,7,8,9}
num_set.add(10)
num_set.remove(10)
flag = 10 in num_set
print(flag)

#字典
people_dict = {'name':'zhangsan','age':18,'sex':'male'}
print(people_dict['name'])
for key,value in people_dict.items():
    print(key,value)
student = {'name':'lisi',19:23}
print(student)
```

###  5.推导式

* 列表推导式

```python
变量名 = [表达式 for 变量 in 列表]
变量名 = [表达式 for 变量 in 列表 if 条件]
变量名 = [表达式 for 变量 in 列表 for 变量 in 列表]
```

### 6.Function

* 什么是函数

所谓的函数就是一个被命名的、独立的、完成特定功能的代码段（一段连续的代码），并可能给调用它的程序一个返回值。

被命名的：在Python中，函数大多数是有名函数（普通函数）。当然Python中也存在没有名字的函数叫做匿名函数。

独立的、完成特定功能的代码段：在实际项目开发中，定义函数前一定要先思考一下，这个函数是为了完成某个操作或某个功能而定义的。（函数的功能一定要专一）

返回值：很多函数在执行完毕后，会通过return关键字返回一个结果给调用它的位置。

```
def 函数名称(参数1, 参数2, ...):
    函数体
    ...
    [return 返回值]


注意: 语法中的中括号表示的可选的意思
```

```Python
def add(a,b):
    return a+b
print(add(1,2))
```

