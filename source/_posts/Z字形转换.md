---
title: Z字形转换
date: 2018-04-10 09:24:54
tags:
- 字符串
- LeetCode
categories:
- 算法
---

## 题目描述
将字符串 "PAYPALISHIRING" 以Z字形排列成给定的行数：（下面这样的形状）
```
P   A   H   N
A P L S I I G
Y   I   R
```
之后按逐行顺序依次排列："PAHNAPLSIIGYIR"



实现一个将字符串进行指定行数的转换的函数:
```
string convert(string text, int nRows);
```

convert("PAYPALISHIRING", 3) 应当返回 "PAHNAPLSIIGYIR" 。

## 解析
考虑convert('abcdefghijklmnop', 4)的情况.用序号代表没个字符的索引, 那么 按照对应的规则排列结果如下
![](http://p3euxxfa8.bkt.clouddn.com/80b5a163371647964aa541aff6efc3ce.png)

和例子给出nRows为3的情况可以看到, 不论行数为多少, 最终都可以如下分类:
![](http://p3euxxfa8.bkt.clouddn.com/8c7e7c395717b484311ebc36725ac04a.png)

因为只是需要读出其中的字母, 中间的空格多少可以忽略, 进一步可抽象为下图
![](http://p3euxxfa8.bkt.clouddn.com/409d22ab6c409c6de1923398271c051b.png)

这样分类每一个区域可以填充的字母数为
```
nRows + nRow -2
```
每块大小为size, 则
```py
size = numRows * 2 - 2
```

所需要的方块数为
```py
count = -1
    length = len(s)
    if length % size == 0:
        count = length / size
    else:
        count = length / size + 1
```

对于上图中的第三个方块, 因为缺少两个元素 , 可以用两个空格表示. 同样需要注意的是, 在每个方块的第二部分是逆序的. 按照这个规则, 字母的排列代码如下:
```py
s += (size - length % size) * ' '  # 补充缺少的元素
matrix = []
# 取得每一块的字母
for i in range(int(count)):
    matrix.append(s[size*i:size*i+size])

# 然后按照规则排列
new_matrix = []
for item in matrix:
    list1 = list(item[:numRows])
    list2 = list(item[numRows:size].center(numRows))[::-1]
    new_matrix.append(list1)
    new_matrix.append(list2)
```

这样就形成了一个符合要求的二维数组, 按行读出即可.
```py
s1 = ''
for i in range(numRows):
    for j in new_matrix:
        s1 += j[i]

return s1.replace(' ', '')
```

同时注意考虑边界情况. 最终代码如下
```py
def convert(s, numRows):
    """
    :type s: str
    :type numRows: int
    :rtype: str
    """

    # 每块的大小size = numRows + numRows -2
    size = numRows * 2 - 2
    if size == 0:
        return s

    # 需要生成的块数为
    count = -1
    length = len(s)
    if length % size == 0:
        count = length / size
    else:
        count = length / size + 1

    s += (size - length % size) * ' '
    matrix = []
    for i in range(int(count)):
        matrix.append(s[size*i:size*i+size])
        # print(matrix)
    new_matrix = []
    for item in matrix:
        list1 = list(item[:numRows])
        list2 = list(item[numRows:size].center(numRows))[::-1]
        new_matrix.append(list1)
        new_matrix.append(list2)
    s1 = ''
    for i in range(numRows):
        for j in new_matrix:
            s1 += j[i]

    return s1.replace(' ', '')
```
