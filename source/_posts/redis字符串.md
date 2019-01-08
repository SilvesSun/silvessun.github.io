---
title: redis字符串
date: 2018-12-18 09:29:41
tags:
- 基本数据结构
- 字符串
categories:
- redis
---

# 命令
`set key value [ex seconds] [px milliseconds] [nx|xx] `

- ex seconds 为键设置秒级过期时间
- px milliseconds 为键设置毫秒级过期时间
- nx|xx nx键必须不存在时才能添加, 用于增加; xx键必须存在才能设置, 用于更新


