---

title: collections
date: 2018-01-29 17:55:13
tags: 

- collections
- build-in module
categories: Python

---

Python 在内置的数据类型的基础上, 在collections模块中提供了几个额外的数据类型. 分别是

- defaultdict
- namedtuple
- deque
- Counter
- UserDict
- UserList
- UserString
- OrderedDict
- ChainMap

## defaultdict

重点关注这一句:

> The first argument provides the initial value for the default_factory attribute; it defaults to None. All remaining arguments are treated the same as if they were passed to the dict constructor, including keyword arguments.

defaultdict 的第一个参数为默认的工厂方法. 使用示例:

```py
s = [('yellow', 1), ('blue', 2), ('yellow', 3), ('blue', 4), ('red', 1)]
d = defaultdict(list)
for k, v in s:
    d[k].append(v)
```

输出

> defaultdict(<class 'list'>, {'yellow': [1, 3], 'blue': [2, 4], 'red': [1]})

另一个常见的用法是, 当对一个键进行嵌套赋值时, 会触发keyError异常。 defaultdict可以解决这个问题

**Question**

```py
s = {}
s['a']['b'] = 2
```

输出

> KeyError: 'a'

**Solution**

```py
d = lambda: collections.defaultdict(d)
s = d()
s['a']['b'] = 'c'
print(s)
```

输出

> defaultdict(<function <lambda> at 0x0000000002072E18>, {'a': defaultdict(<function <lambda> at 0x0000000002072E18>, {'b': 'c'})})

## namedtuple

把元组变成一个针对简单任务的容器。你不必使用整数索引来访问一个namedtuples的数据。你可以像字典(dict)一样访问namedtuples，但namedtuples是不可变的。

签名:

```py
def namedtuple(typename, field_names, *, verbose=False, rename=False, module=None)
```

一个命名元组(namedtuple)有两个必需的参,它们是元组名称和字段名称

使用namedtuple的好处是可以使用名称来访问, 同时可以将其转换为字典

```py
from collections import *

Animal = namedtuple('Animal', 'name age')
dog = Animal('Husky', 5)

print(dog.age)

# convert to dict
print(dog._asdict())
```

输出:

> 5
> 
> OrderedDict([('name', 'Husky'), ('age', 5)])

**Question**
现在有一txt文件, 分别对应商品的名称, 尺码, 价格, 数量. 然后将每一条记录构建为namedtuple

**Answer**
在工作中遇到这个问题的情景是涉及到的字段很多, 接近50个. 这种情况下如果用索引来读取信息会显得繁琐且容易出错. 换成namedtuple可以优美的解决问题

```py
# 列出所有的字段
headers = ['name', 'size', 'price', 'qty']
# 构建一个namedtuple

Row = namedtuple('Row', headers)

# 然后每一个读取到的值就可以构建为可以类似属性访问的操作了
with open(filename, 'rb') as f:
    reader = f.readlines()
    for i, data_str in enumerate(reader):
        data_list = data_str.split('|')
        data = Row._make(data_list)

        print(data.name)
```

这里的_make()的作用是

> 'Make a new named tuple object from a sequence or iterable.'

## deque

双端队列.

## Counter

计数器, 可以针对某项数据进行计数.

## OrderedDict

Python中的字典本身是无序的, 当需要保持字典有序的时候, 可以使用OrderedDict
