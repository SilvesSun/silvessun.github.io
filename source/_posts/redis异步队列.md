---
title: redis异步队列
date: 2018-09-06 10:55:41
tags:
- 队列
- 列表
- 有序集合
categories:
- redis
---

## 1.列表
列表的特点: 有序, 允许重复的元素

列表命令:

|操作|命令|
|---|---|
|添加|rpush lpush linsert|
|查|lrange lindex llen|
|删除|lpop rpop lrem ltrim|
|修改|lset|
|阻塞操作|blpop brpop|

## 2.异步消息队列
使用列表作为异步消息队列, 使用rpush/lpush操作入队列，使用lpop 和 rpop来出队列。

### 2.1 队列为空的处理
如果队列为空, 就会陷入pop的死循环. 这就是空轮询, 空轮询不但会浪费客户端的CPU, 还会拉高redis的QPS, 影响查询性能

解决这个问题的办法是让线程睡眠一段时间, 一般为1s. 或者使用blpop或brpop, 这样一有消息就会处理

## 延时队列
延时队列可以通过 Redis 的 zset(有序列表) 来实现。我们将消息序列化成一个字符串作为 zset 的value，这个消息的到期处理时间作为score，然后用多个线程轮询 zset 获取到期的任务进行处理
