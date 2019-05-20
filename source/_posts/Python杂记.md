---
title: Python杂记
date: 2019-03-18 09:40:41
tags:
- 释疑
- 面试
categories:
- Python
---

# 1.Python的可变对象和不可变对象
诸如（int，float，bool，str，tuple，unicode）等内置类型的对象是不可变的。像（list，set，dict）这样的内置类型的对象是可变的

下面代码的输出为?

```py 
s = "hello"
s[2] = "u"
```

因为 Python 中字符串是不可变对象, 所以不允许修改字符串的值. 输出为

```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'str' object does not support item assignment
```

# 2. == 和 is 的区别

- `==` 比较两个对象的值, 对象中保存的数据, is 比较的是两个对象的标识
- is 比 == 要更快, 因为is不能重载, 所以 Python 不用寻找并调用特殊方法, 而是直接比较两个对象的ID

# 3.迭代器, 生成器, 可迭代对象
## 可迭代对象
可以返回一个迭代器的对象可称之为可迭代对象. 可以使用 for 来循环遍历的对象. 可迭代对象具有 `__iter__` 方法, 用于返回一个迭代器; 或者定义了`getitem`
方法, 可以按照index索引的对象.

## 迭代器对象

迭代器协议

```
指要实现对象的 __iter()__ 和 next() 方法（注意：Python3 要实现 __next__() 方法），其中，__iter()__ 方法返回迭代器对象本身，next() 方法返回容器的下一个元素，在没有后续元素时抛出 StopIteration 异常
```

迭代器是指遵循迭代器协议（iterator protocol）的对象

## 生成器
生成器就是一种迭代器。生成器拥有next方法并且行为与迭代器完全相同，这意味着生成器也可以用于Python的for循环中

生成器使用 `yield` 返回值


# 协程
协程是比线程更轻量的存在. 一个线程也可以拥有多个协程. 协程由用户程序控制. yield让协程暂停，和线程的阻塞是有本质区别的。协程的暂停完全由程序控制，线程的阻塞状态是由操作系统内核来进行切换。
因此，协程的开销远远小于线程的开销. 
