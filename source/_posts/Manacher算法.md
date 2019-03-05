---
title: Manacher算法
date: 2019-01-16 15:06:03
tags:
- 字符串
- 回文字符串
categories:
- 算法
---

`Manacher算法`是为了改进回文字符串查找过程中重复访问字符串这一缺陷而使用的算法, 由一个叫Manacher的人在1975年发明. 其基本思路是在原字符串上添加额外的
字符来避免对奇偶回文字符串的判断, 以及利用回文中心降低访问字符串的频率, 将算法复杂度降低到 `O(n)`的算法

# 字符串预处理
在所有的空隙位置(包括首尾)插入同样的符号，要求这个符号是不会在原串中出现的。这样会使得所有的串都是奇数长度的. 同时为了防止越界, 在字符串开头和结尾也添加
特殊字符
```
aba  ———>  #a#b#a#
abba ———>  #a#b#b#a#
```

# 基本思想
定义`回文半径`, 回文串中最左或最右位置的字符与其对称轴的距离。算法中以`RL`表示. 我们在这里用一个长度为经过处理的字符串的长度的数组RL表示每个不同字符位置
所对应的最大的回文字符串半径

定义 `MaxRight`，表示当前访问到的所有回文子串，所能触及的最右一个字符的位置, 将`MaxRight`对应的回文串的对称轴所在的位置，记为`pos`

遍历字符串的过程中, 将当前位置记为`i`. 

## i < MaxRight 的两种情况
### MaxRight - i > RL[2*pos-i]

![MaxRight - i > RL[2*pos-i]](http://supcoder.net/MaxRightgti1.png)

如上图所示, 以S[j]为中心的回文子串包含在以S[id]为中心的回文子串中，由于 i 和 j 对称，以S[i]为中心的回文子串必然包含在以S[id]为中心的回文子串中，所以必有 P[i] = P[j]

### MaxRight - i ＜ RL[2*pos-i]
![MaxRightgti2](http://supcoder.net/MaxRightgti2.png)
以S[j]为中心的回文子串不一定完全包含于以S[id]为中心的回文子串中，但是基于对称性可知，下图中两个绿框所包围的部分是相同的，也就是说以S[i]为中心的回文子串，其向右至少会扩张到mx的位置，也就是说 P[i] >= mx - i

## i ＞ MaxRight
无法对 P[i]做更多的假设，只能P[i] = 1，然后再去慢慢匹配了

# 代码实现
```py
def manacher(s):
    s_new = "".join(("$#", "#".join(s), "#"))
    p = [0]
    center = 0
    mx = 0
    for i in range(1, len(s_new)):
        # i ＞ MaxRight
        if i > mx:
            p.append(0)
        
        # 取 min(RL[2*pos-i], MaxRight-i)
        else:
            p.append(min(mx - i, p[2 * center - i]))

        # 逐步匹配的过程    
        while (i - p[i] - 1) > 0 and (i + p[i] + 1) < len(s_new) and s_new[i - p[i] - 1] == s_new[i + p[i] + 1]:
            p[i] += 1
        if p[i] > mx - center:
            center = i
            mx = i + p[i]
    return p


```