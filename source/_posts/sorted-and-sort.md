---
title: sorted and sort
date: 2018-01-30 11:29:24
tags: Python基础
categories:
- Python
---

Python 内置的可以用来排序的函数有sort 和 sorted.
- list.sort() 是直接在原列表的基础上排序, 返回值为None
- sorted 相较于sort的功能更加强大, 使用范围要更加广泛. sorted接受一切迭代器, 返回新的列表.

常见的用到排序的比如字典排序
- 依据key排序

```py
>>>s = {1: 'D', 2: 'B', 3: 'C', 4: 'E', 5: 'A'}
>>>sorted(s.items(), key=lambda x: x[0])
```

- 依据value排序
```py
>>>s = {1: 'D', 2: 'B', 3: 'C', 4: 'E', 5: 'A'}
>>>sorted(s.items(), key=lambda x: x[1])
```

在这里其实是用s.items()将字典转化为元组组成的列表, 然后依据元组的元素排序.
