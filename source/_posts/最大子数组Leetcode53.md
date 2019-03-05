---
title: 最大子数组Leetcode53
date: 2019-02-15 11:09:49
tags:
- 动态规划
- 分冶
- LeetCode
categories:
- 算法
---

## 问题
Given an integer array nums, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum.

Example:
```
Input: [-2,1,-3,4,-1,2,1,-5,4],
Output: 6
Explanation: [4,-1,2,1] has the largest sum = 6.
```

## 动态规划
**Kadane算法思路**

Kadane算法将子问题定义为：求以第i个元素结尾的最大子数组

假设有数组A:[1, -3, 2, 4, -5]，则：

- 以第0个元素结尾的最大子数组就是它自己：[1]
- 以第1个元素结尾的最大子数组可能是这些中的一个：[1, -3], [-3]
- 以第2个元素结尾的最大子数组可能是这些中的一个：[1, -3, 2], [-3, 2], [2]

> 以第i个元素结尾的子数组，一定是以第i-1个元素结尾的子数组加上第i个元素组成的

故有:
```
maxSum[0] = A[0]; // i = 0
maxSum[i] = A[i] + （maxSum[i-1] > 0 ? maxSum[i-1] : 0); // i > 0
```

所以如果以第i-1个元素结尾的最大子数组和大于0，那么以第i个元素结尾的最大子数组就等于它前面的那部分加上它自己，否则就舍弃前面的部分，只剩它自己

maxSum这个数组中的每个元素就代表了以当前位置结尾的最大子数组和，取maxSum中的最大值

### 代码
```py
def maxSubArray(self, nums):
    maxSum = [None for _ in range(len(nums))]
    maxSum[0] = nums[0]
    gMax  = nums[0]
    for i in range(1, len(nums)):
        maxSum[i] = nums[i] + max(0, maxSum[i-1])
        gMax = max(gMax, maxSum[i])
    return gMax
```

#### 优化
实时更新最大值, 而不用使用数组来存储

```py
def maxSubArray(self, nums):
    """
    :type nums: List[int]
    :rtype: int
    """
    if not nums:
        return 0
    current = nums[0]
    gMax = current
    for i in range(1, len(nums)):
        if current < 0:
            current = 0
        current += nums[i]
        gMax = max(current, m)
    return gMax

```

## 分冶法
针对给定的一个区段，首先将它划分成相等的两个部分。这个最大连续子串可能出现在它的左边子串里，也可能出现在它的右边子串里，
也有可能是通过跨越左右两个子串的这个串。所以我们要计算出它们两个部分，取它们中间最大的那个.

前两种情况只需递归调用即可；第三种情况需要单独处理。需要从 mid 开始分别向两个方向不断求和，分别计算出两个方向的最大值（两者的端点其中之一均为中间点），求和就得到第三种情况的最大值

```python
def maxSubArray2(self, nums):
    """
    分冶法
    :param nums:
    :return:
    """
    return self.helper(nums, 0, len(nums) - 1)

def helper(self, nums, left, right):
    if left == right: return nums[left]
    mid = (left + right) / 2

    left_max = sum_num = 0
    for i in range(mid - 1, left - 1, -1):
        sum_num += nums[i]
        left_max = max(left_max, sum_num)

    right_max = sum_num = 0
    for i in range(mid + 1, right + 1):
        sum_num += nums[i]
        right_max = max(right_max, sum_num)

    left_ans = self.helper(nums, left, mid - 1)
    right_ans = self.helper(nums, mid + 1, right)

    return max(left_max + nums[mid] + right_max, max(left_ans, right_ans))
```
