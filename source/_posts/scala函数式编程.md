---
title: scala函数式编程
date: 2018-08-22 09:53:58
tags:
- 函数式
categories:
- scala
---

函数也是值, 可以赋值给一个变量, 存储在数据结构或者作为参数传递给另一个函数

## 高阶函数

把一个函数当做参数传递给另一个函数. 这样的函数叫做高阶函数

```scala
def formatResult(name:String, n:Int, f:Int => Int) = {
    val msg = "The %s of %d is %d"
    msg.format(name, n, f(n))
}
```

这里的 formatResult 就是一个高阶函数, 接受一个 f 函数作为参数. 这个函数 f 的类型是 Int => Int, 表示接收一个整型参数并返回一个整型结果


## 多态函数

函数式编程的多态意味着`参数化多态`. 

```scala
def findFirst[A](as: Array[A], p: A => Boolean): Int = {
    @annotation.tailrec
    def loop(n: Int):Int =
        if (n >= as.length) -1
        else if (p(as(n))) n
        else loop(n+1)
    loop(0)
}
```

这就是一个多态函数. 将逗号分隔的类型参数, 紧跟在函数名称后面使用中括号括起来. 

调用foreach组合器并不会在计算值可用的时候阻塞当前的进程去获取计算值。恰恰相反，只有当future对象成功计算完成了，foreach所迭代的函数才能够被异步的执行。这意味着foreach与onSuccess回调意义完全相同

如果我们想让我们的Future对象返回0而不是抛出那个该死的异常，那我们需要使用recover组合器