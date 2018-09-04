---
title: golang标准库之sync
date: 2018-06-26 10:34:18
tags:
- go标准库
- sync
categories:
- golang学习
---

![go图标](http://p3euxxfa8.bkt.clouddn.com//18-6-26/73283110.jpg)

今天学习golang标准库中的sync包.

在源码中, sync的文件结构如下图所示:

![sync文件结构](http://p3euxxfa8.bkt.clouddn.com/2018-06-26-10-39-39.png)

可以看到, sync中包含一个package(atomic) 和 cond, mutex, once, pool, waitgroup等包. 其中atomic提供一些底层的原子操作, 一般情况下不建议使用, 故在这里也不多说.

# cond
这个类用于goroutine之间进行协作, 包括三个函数, Broadcast() , Signal(), Wait(); 一个成员变量，L　Lock.

Broadcast() 的动能是*唤醒在这个cond上等待的所有的goroutine*, Signal()选择其中一个唤醒, Wait是让 goroutine 等待

# Mutex / RWMutex
互斥锁, 保证任意时刻只有一个例程访问某个对象

# Once
Once 的作用是多次调用但只执行一次，Once 只有一个方法，Once.Do()，向 Do 传入一个函数，这个函数在第一次执行 Once.Do() 的时候会被调用，以后再执行 Once.Do() 将没有任何动作，即使传入了其它的函数，也不会被执行，如果要执行其它函数，需要重新创建一个 Once 对象

# pool
用于存储临时对象，它将使用完毕的对象存入对象池中，在需要的时候取出来重复使用，目的是为了避免重复创建相同的对象造成 GC 负担过重。其中存放的临时对象随时可能被 GC 回收掉（如果该对象不再被其它变量引用）

# runtime
Go的runtime负责对goroutine进行管理。所谓的管理就是“调度”，粗糙地说调度就是决定何时哪个goroutine将获得资源开始执行、哪个goroutine应该停止执行让出资源、哪个goroutine应该被唤醒恢复执行等

# WaitGroup
WaitGroup 用于等待一组例程的结束。主例程在创建每个子例程的时候先调用 Add 增加等待计数，每个子例程在结束时调用 Done 减少例程计数。之后，主例程通过 Wait 方法开始等待，直到计数器归零才继续执行。

