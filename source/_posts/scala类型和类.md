---
title: scala类型和类
date: 2018-09-12 11:11:33
tags:
- class
- type
categories:
- scala类型系统
---

![](http://p3euxxfa8.bkt.clouddn.com//18-9-12/56872846.jpg)

## class 和 type

scala 有一个统一的类型系统. 如图所示
![](http://p3euxxfa8.bkt.clouddn.com//18-9-12/42518128.jpg)

通过 Any 这个类, 将对应于对象的 AnyRef 和对应于值的 AnyVal 统一起来. 以及一个底类型 Nothing, 形成了统一的系统类型.

对于一个类, 可以通过 `typeOf` 获取类型信息, 通过 `classOf` 获取类信息.

```scala
scala> import scala.reflect.runtime.universe._

scala> class A

scala> classOf[A]

res0: Class[A] = class A

scala> typeOf[A]
res1: reflect.runtime.universe.Type = A

```

可以看到, 在scala中类与类型是两个不一样的概念. 类型比类更具体, 任何数据都有类型. 类型一致的对象其类也是一致的, 但是类一致的其类型不一定一致

```scala
scala> classOf[List[Int]] == classOf[List[String]]
res16: Boolean = true

scala> typeOf[List[Int]] == typeOf[List[String]]
res17: Boolean = false
```


