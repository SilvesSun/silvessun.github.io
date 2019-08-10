---
title: 使用lxml读xml文档
date: 2018-03-01 10:00:54
tags:
- lxml
- xml
- Python模块i
categories:
- Python
---

最近在做项目的过程中需要解析客户发过来的xml文件, 之前一般使用的都是xml.etree.ElementTree, 趁这个机会学习下新的xml处理模块-----lxml

## 读取xml文档
1. lxml可以从xml字符串中解析xml文档.
```py
from lxml import etree
text = r'<xml><head></head><body></body></xml>'
xml = etree.fromstring(text)
```

这样生成的是一个lxml.etree.\_Element 对象.

2. 从文件中读取.
```py
xml = etree.parse(XML_NAEM)
```

## 获取节点属性
这里以一下xml文件为例
```xml
<?xml version="1.0"?>
<catalog version="1.0">
   <book id="bk101">
      <author>Gambardella, Matthew</author>
      <title>XML Developer's Guide</title>
      <genre>Computer</genre>
      <price>44.95</price>
      <publish_date>2000-10-01</publish_date>
      <description>An in-depth look at creating applications
      with XML.</description>
   </book>
   <book id="bk102">
      <author>Ralls, Kim</author>
      <title>Midnight Rain</title>
      <genre>Fantasy</genre>
      <price>5.95</price>
      <publish_date>2000-12-16</publish_date>
      <description>A former architect battles corporate zombies,
      an evil sorceress, and her own childhood to become queen
      of the world.</description>
   </book>
</catalog>
```

```py
# 获取根节点
xml_root = xml.getroot()

# 根节点的所有属性
print(xml_root.items())

# 获取对应的属性
print(xml_root.get('version', ''))

#获取根节点的子节点
children = root.getchildren()
```

结果
```
[<Element book at 0x2697f88>, <Element book at 0x2697f48>]
```

可以看到getchildren返回的是一个子元素构成的列表, 同时将每个子节点的label直接展示出来.

同时也可以取得某个具体的节点的属性及值
```py
for child in children[0]:
    print child.tag, child.text
```

输出
```
('author', 'Gambardella, Matthew')
('title', "XML Developer's Guide")
('genre', 'Computer')
('price', '44.95')
('publish_date', '2000-10-01')
('description', 'An in-depth look at creating applications \n      with XML.')
```

如果对文档结构熟悉的情况下, 也可以用find获得对应信息. 比如:
```py
xml_book = root.find('./book')
print(xml_book)

# 结果为<Element book at 0x26c7f48>, 这里find返回的是第一个符合条件的元素, 如果要获得所有的元素, 使用findtext

print(root.findall('./book'))

# 如果要获取某个具体的属性值:
print(root.findtext('./book/author'))  # 结果Gambardella, Matthew, 也就是第一个满足条件的标签值.

```

另外xpath也是一种可选方案:
```py
print root.xpath('./book')  # [<Element book at 0x26f7f88>, <Element book at 0x26f7f48>],
```

附xpath对应常用表达:

|    路径表达式    | 结果 |
| ---------- | --- |
| book |  选取book的所有子节点 |
| /book       |  根元素book |
| catalog/book | 选取属于catalog的所有book元素|
| //book   | 选取文档中的所有book元素  |
