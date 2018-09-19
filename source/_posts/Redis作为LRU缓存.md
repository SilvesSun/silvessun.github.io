---
title: Redis作为LRU缓存
date: 2018-09-04 11:30:23
tags:
- 缓存
- LRU
- 翻译
categories:
- redis
---

![](http://p3euxxfa8.bkt.clouddn.com//18-9-4/97767057.jpg)

此文基于[Using Redis as an LRU cache](https://redis.io/topics/lru-cache)

当 redis 作为缓存使用时, 当有内存限制的时候, 添加数据的时候, 需要淘汰一部分旧的数据. 

LRU(Least Recently Used) 是 redis 支持的内存回收策略的一种. 本文将要讲述用于配置redis内存大小的`maxmemory`配置选项, 并深入讨论 LRU(确切的说只是近似实现) 在redis中的使用.

## maxmemory 配置选项
maxmemory 配置选项使用来配置 Redis 的存储数据所能使用的最大内存限制. 可以通过在内置文件redis.conf中配置，也可在Redis运行时通过命令CONFIG SET来配置。例如，我们要配置内存上限是100M的Redis缓存，那么我们可以在 redis.conf 配置如下：
```
maxmemory 100mb
```

当 maxmemory 设置为0表示没有内存限制. 在64位系统中, 默认0为无限制, 而在32位系统中, 默认是3GB

当存储的数据到达内存上限时, 如果有新的数据, redis 会基于不同的策略, 来决定是直接返回错误还是清楚部分老的数据而使新的数据能够加入.

## 回收策略
可用的策略如下:
- `noenviction`：不清除数据，只是返回错误，这样会导致浪费掉更多的内存，适用于大多数写命令（DEL 命令和其他的少数命令例外）

- `allkeys-lru`：从所有的数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰，以供新数据使用

- `volatile-lru`：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰，以供新数据使用

- `allkeys-random`：从所有数据集（server.db[i].dict）中任意选择数据淘汰，以供新数据使用

- `volatile-random`：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰，以供新数据使用

- `volatile-ttl`：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰，以供新数据使用

redis使用 `maxmemory-policy`配置具体的回收策略

当缓存中没有符合条件的键时, volatile-lru, volatile-random 和volatile-ttl 将会和 noeviction 一样返回错误. 正确的选择回收策略取决于应用程序的访问模式. 可以在程序运行时, 根据`INFO`命令的监控结果, 动态的选择回收策略, 从而调优配置.

根据使用经验, 一般来说回收策略可以这样来配置:
- 如果期望用户请求呈现幂律分布(power-law distribution)，也就是，期望一部分子集元素被访问得远比其他元素多时，可以使用allkeys-lru策略。在你不确定时这是一个好的选择

- 如果期望是循环周期的访问，所有的键被连续扫描，或者期望请求符合平均分布(每个元素以相同的概率被访问)，可以使用allkeys-random策略。

- 如果你期望能让 Redis 通过使用你创建缓存对象的时候设置的TTL值，确定哪些对象应该是较好的清除候选项，可以使用volatile-ttl策略。

另外值得注意的是，为键设置过期时间需要消耗内存，所以使用像allkeys-lru这样的策略会更高效，因为在内存压力下没有必要为键的回收设置过期时间。

