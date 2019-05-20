---
title: "\x90Python类装饰器"
date: 2019-05-13 16:32:48
tags:
categories:
---

# 可调用对象
在Python中, 除开函数外, 调用运算符(`()`) 还可以应用到其他的对象上. 可以使用内置函数`callable()` 来判断对象是否可被调用. 任何对象都可以表现的像
Python函数, 只要定义了 `__call__`. 所以也可以用类来实现一个装饰器

# 类装饰器

实现一个函数执行时间的类装饰器

```Python
class Timer(object):
  def __init__(self, func):
    self.func = func

  def __call__(self, *args, **kwargs):
    start = time.now()
    res = self.func(*args, **kwargs)
    after = time.now()
    print(after-start)
    return res

@Timer
def add(x, y):
  return x+y
```



