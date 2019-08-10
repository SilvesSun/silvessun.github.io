---
title: redis之HyperLogLog
date: 2018-12-27 12:00:32
tags:
- 基数统计
categories:
- redis
---

Redis HyperLogLog 是用来做基数统计的算法. HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定的、并且是很小的。

在Redis里面，每个Hyperloglog键只需要12Kb的大小就能计算接近2^64个不同元素的基数. 但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素

这里有一个新的概念: 基数统计.

## 基数
比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数

## 用途
可以非常省内存的去统计各种计数，比如注册ip数、每日访问IP数、页面实时UV（PV肯定字符串就搞定了）、在线用户数

## 使用方法

redis 提供 pfadd 和 pfcount 两个指令, 分别用于增加计数和获取计数

### pfadd
PFADD key element [element ...] 添加指定元素到 HyperLogLog 中

如果 HyperLogLog 估计的近似基数（approximated cardinality）在命令执行之后出现了变化， 那么命令返回 1 ， 否则返回 0 。 如果命令执行时给定的键不存在， 那么程序将先创建一个空的 HyperLogLog 结构， 然后再执行命令。

```shell
127.0.0.1:6379> PFADD test user1
(integer) 1
127.0.0.1:6379> pfadd test user2
(integer) 1
127.0.0.1:6379> pfadd test user1
(integer) 0
127.0.0.1:6379>

```

可以看到第二次向test中添加user1 后, 由于近似基数没有发生变化, 所以返回值为0

### pfcount
PFCOUNT key [key ...]

当 PFCOUNT 命令作用于单个键时， 返回储存在给定键的 HyperLogLog 的近似基数， 如果键不存在， 那么返回 0 。

当 PFCOUNT 命令作用于多个键时， 返回所有给定 HyperLogLog 的并集的近似基数， 这个近似基数是通过将所有给定 HyperLogLog 合并至一个临时 HyperLogLog 来计算得出的

```shell
127.0.0.1:6379> pfcount test
(integer) 2

```

## pf 的内存占用为什么是 12k？
在 Redis 的 HyperLogLog 用到的是 16384 个桶，也就是 2^14，每个桶的 maxbits 需要 6 个 bits 来存储，最大可以表示 maxbits=63，于是总共占用内存就是2^14 * 6 / 8 = 12k字节。



