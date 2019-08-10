---
title: Django自定义tag
date: 2018-09-03 10:11:52
tags:
- Django
- tag
categories:
- web
---

Django 提供了一系列的內建的模板标签来完成模板中基本的显示, 但在实际的使用过程中不可避免的会出现 Django 提供的标签不能满足现有需求的情况. 这个时候就需要自定义标签

建立一个自定义标签, 首先需要在当前 app 目录下新建一个`templatetags` 模块. 

Django 提供的自定义标签包括三种

 - **simple_tag**: 返回值为字符串

 - **inclusion_tag**: 返回一个渲染过的模板

 - **assignment_tag**: 设置请求上下文


示例
```py
from django import template
@register.simple_tag()
def muti(*args):
    if args:
        return reduce(lambda x, y: x * y, list(args))
```
