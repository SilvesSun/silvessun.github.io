---
title: scala结构类型
date: 2018-09-14 09:35:23
tags:
categories:
- scala类型系统
---

在 scala 中, 可以通过 type 关键字声明为 **结构类型**

```scala
type X = { def close():Unit }

```

这样当需要约束参数实现了 close 方法的时候, 只需要这样写:

```scala
def f(res:X) = res.close

```
