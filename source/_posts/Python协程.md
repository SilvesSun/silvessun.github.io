---
title: Python协程
date: 2019-03-18 15:41:27
tags:
- 协程
- 读书笔记
- 流畅的Python
categories:
- Python
---

Python2.5 以后, yield 关键字可在表达式中使用, 并在生成器的API中添加了 `.send(value)` 方法. 生成器的调用方可以使用`.send(...)` 发送数据, 发送的数据
会成为生成器函数中 `yield` 表达式的值. 

协程是指一个过程, 这个过程与调用方协作, 产出由调用方提供的值

除开 send() 外, pep342 中还定义了 `.throw(...)` 用于调用方抛出异常, 在生成器中处理; `.close()` 终止生成器

```Python
>>>def simple_coroutine():
    print("start coroutine")
    x = yield
    print("recieve value:", x)
    
>>>coro = simple_coroutine()
>>>coro
<generator object simple_coroutine at 0x0000000003D47570>
>>>next(coro)
start coroutine
>>>coro.send(42)  ## 调用 send 后, yield表达式计算出 42. 协程恢复, 一直运行到下一个yield表达式, 或者终止
Traceback (most recent call last):
  File "<input>", line 1, in <module>
StopIteration
recieve value: 42
```

激活一个协程可以使用

```Python
>>>next(coro)
start coroutine

# or
>>>coro.send(None)
start coroutine
```


如果创建协程对象后不激活协程, 将 None 以外的值传递给它, 将会报错

```Python
>>>coro = simple_coroutine()
>>>coro.send(5)
Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: can't send non-None value to a just-started generator

```

另一个例子

```Python
def simple_coro2(a):
    print("-> start a = ", a)
    b = yield a  ## = 右边的代码先执行, 因此要在客户端代码再激活时才会复制给b
    print("-> received b = ", b)
    c = yield a + b
    print("-> received c=", c) 
coro2 = simple_coro2(14)
```

```shell
>>>next(coro2)
-> start a =  14
14

>>>coro2.send(28)
-> received b =  28
42

>>> coro2.send(99)
-> received c= 99
Traceback (most recent call last):
  File "<input>", line 1, in <module>
StopIteration
```

所有的协程都需要预激才能使用, 使用 `yield from` 语法时, 会自动预激. 

## yield from
`yield from x` 表达式对x对象做的事情为:

1. 调用 iter(x) 从中获取迭代器. x是任何可迭代的对象
2. 打开双向通道, 把最外层的调用方与最内层的子生成器连接起来, 这样两者可以直接发送和产出值, 传入异常

###术语
`委派生成器` 包含 yield from <Iterable> 表达式的生成器函数

`子生成器` 从yield from 表达式中, <Iterable> 部分获取的生成器

`调用方` 调用委派生成器的客户端代码

```py 
# 子生成器
def averager():  # <1>
    total = 0.0
    count = 0
    average = None
    while True:
        term = yield  # <2>
        if term is None:  # <3>
            break
        total += term
        count += 1
        average = total/count
    return Result(count, average)  # <4>


# 委派生成器
def grouper(results, key):  # <5>
    while True:  # <6>
        results[key] = yield from averager()  # <7>


# 调用方
def main(data):  # <8>
    results = {}
    for key, values in data.items():
        group = grouper(results, key)  # <9>
        next(group)  # <10> 
        for value in values:
            group.send(value)  # <11>
        group.send(None)  # important! <12>

    # print(results)  # uncomment to debug
    report(results)


# output report
def report(results):
    for key, result in sorted(results.items()):
        group, unit = key.split(';')
        print('{:2} {:5} averaging {:.2f}{}'.format(
              result.count, group, result.average, unit))


data = {
    'girls;kg':
        [40.9, 38.5, 44.3, 42.2, 45.2, 41.7, 44.5, 38.0, 40.6, 44.5],
    'girls;m':
        [1.6, 1.51, 1.4, 1.3, 1.41, 1.39, 1.33, 1.46, 1.45, 1.43],
    'boys;kg':
        [39.0, 40.8, 43.2, 40.8, 43.1, 38.6, 41.4, 40.6, 36.3],
    'boys;m':
        [1.38, 1.5, 1.32, 1.25, 1.37, 1.48, 1.25, 1.49, 1.46],
}
```


