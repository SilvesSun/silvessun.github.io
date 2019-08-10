---
title: Cookie与Session的区别
date: 2018-03-07 16:07:05
tags:
- web
categories:
- web基础
---

## 定义
### Cookie
Cookie技术是客户端的解决方案，Cookie就是由服务器发给客户端的特殊信息，而这些信息以文本文件的方式存放在客户端，然后客户端每次向服务器发送请求的时候都会带上这些特殊的信息。

其实本质上cookies就是http的一个扩展。有两个http头部是专门负责设置以及发送cookie的,它们分别是Set-Cookie以及Cookie

### Session
除了使用Cookie，Web应用程序中还经常使用Session来记录客户端状态。Session是服务器端使用的一种记录客户端状态的机制，使用上比Cookie简单一些，相应的也增加了服务器的存储压力

## 区别

- Cookie数据保存在客户端，Session保存在服务器
- 服务端保存状态机制需要在客户端做标记，所以session可能借助cookie机制，比如服务器返回的sessionId存放的容器就是cookie（注意：有些资料说ASP解决这个问题，当禁掉cookie，服务端的session任然可以正常使用，ASP没试验过，但是对于网络上很多用php和jsp编写的网站，我发现禁掉cookie，网站的session都无法正常的访问）
- Cookie通常用于客户端保存用户的登录状态
- 单个cookie在客户端的限制是3K，就是说一个站点在客户端存放的COOKIE不能超过3K
