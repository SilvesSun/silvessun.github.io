---
title: Python描述符
date: 2019-03-19 15:09:20
tags:
categories:
---

描述符是实现了特定协议的类, 这个协议包括`__get__`, `__set__`, `__del__`

property类实现了完整的描述符协议, 通常可以只实现部分协议. 描述符的用法是创建一个实例, 作为另一个类的类属性

- 覆盖型描述符
实现 `__set__`方法的描述符属于覆盖型描述符

- 非覆盖型描述符

没有实现`__set__`方法的描述符

不管描述符是不是覆盖型, 为类属性赋值都能覆盖描述符, 这是一种猴子补丁技术

## 方法是描述符
在类中定义的方法属于绑定方法, 因为用户定义的函数都有`__get__`方法, 所以依附到类上, 就相当于描述符



