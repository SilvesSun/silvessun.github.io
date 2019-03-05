---
title: pathSum
date: 2019-02-19 14:45:49
tags:
categories:
---

## 问题1
Given a binary tree and a sum, determine if the tree has a root-to-leaf path such that adding up all the values along the path equals the given sum

For example:
Given the below binary tree and sum = 22,

              5
             / \
            4   8
           /   / \
          11  13  4
         /  \      \
        7    2      1
return true, as there exist a root-to-leaf path 5->4->11->2 which sum is 22.

### 思路
在遍历到叶子的过程中顺便判断sum是否等于目标值, 发现不合适的直接返回

### 解题
```py

class Solution(object):
    def hasPathSum(self, root, sum):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: bool
        """
        if not root: return False
        sum -= root.val
        if (not root.left) and (not root.right):
            if sum == 0:
                return True
            else:
                return False
        if root.left and self.hasPathSum(root.left, sum): return True
        if root.right and self.hasPathSum(root.right, sum): return True
        return False
```

## 问题2
You are given a binary tree in which each node contains an integer value.

Find the number of paths that sum to a given value.

The path does not need to start or end at the root or a leaf, but it must go downwards (traveling only from parent nodes to child nodes).

The tree has no more than 1,000 nodes and the values are in the range -1,000,000 to 1,000,000.

Example:

root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8

      10
     /  \
    5   -3
   / \    \
  3   2   11
 / \   \
3  -2   1

Return 3. The paths that sum to 8 are:

```
  1.  5 -> 3
  2.  5 -> 2 -> 1
  3. -3 -> 11
```

### 思路
对于遍历到的节点, 可能的结果是这个节点的值的和满足sum, 也有可能是不包含此节点的值满足sum.故
```py
def pathSum(self, root, sum):
  if not root: return 0
  # 包含本节点的值的和为sum的情况
  ans = traverse(root, sum)

  # 不包含本节点的值的和为sum的情况
  ans += self.pathSum(root.left, sum)
  ans += self.pathSum(root.right, sum)
  return ans
```

然后现在需要找到一条从当前节点到某一节点和为sum的情况. 相较于第一题来说, 只是遍历路径可能不到叶子节点. 故

```py
def traverse(root, val):
    if not root: return 0
    res = (val == root.val)

    res += traverse(root.left, val - root.val)
    res += traverse(root.right, val - root.val)
    return res
```

最终的代码为

```py
class Solution(object):
    def pathSum(self, root, sum):
        """
        :type root: TreeNode
        :type sum: int
        :rtype: int
        """

        def traverse(root, val):
            if not root: return 0
            res = (val == root.val)

            res += traverse(root.left, val - root.val)
            res += traverse(root.right, val - root.val)
            return res

        if not root: return 0
        ans = traverse(root, sum)
        ans += self.pathSum(root.left, sum)
        ans += self.pathSum(root.right, sum)
        return ans
```