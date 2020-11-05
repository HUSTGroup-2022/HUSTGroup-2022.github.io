---
title: 239.滑动窗口的最大值(Hard)(Python)
date: 2020-11-04 10:06:53
categories:
- LeetCode
- 动态规划
- 单调队列
tags:
- LeetCode
- 动态规划
- 单调队列
---
[LeetCode 239](https://leetcode-cn.com/problems/sliding-window-maximum)，滑动窗口最大值，难度：`困难`。

## 问题解析

### 问题描述
给定一个数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 k 个数字。
滑动窗口每次只向右移动一位。返回滑动窗口中的最大值。能否在线性时间内解决这个问题？

More info: [原题](https://leetcode-cn.com/problems/the-skyline-problem/)

### 示例

>输入:  nums = [1,3,-1,-3,5,3,6,7], 和 k = 3

>输出:  [3,3,5,5,6,7]

### 题解

1、首先想到的是暴力解法，滑动窗口，每次在窗口内扫描最大值，这样的复杂度是O(Nk)。

2、题目要求的是线性时间，那么从暴力解法中寻求优化方法。考虑到每次窗口内扫描最大值很耗费时间，则使用一个数据结构来储存当前窗口的最大值。
同时为了保证数据结构中存储的元素不至于超出窗口长度，存储元素的索引来判断是否超出长度。因为每次要取出最大值，所以考虑单调栈。
这个数据结构需要弹出元素，而单调栈默认栈顶是最大值，为了偷懒，使用单调的双端队列，其插入和弹出都是O(1)时间，所以整体上是线性时间。

3、动态规划：参考官方题解。把数组按窗口长度分块，对每个块求两个动态数组————从块开始往块结尾的最大值left和从块结尾往块开始的最大值right。
这样的两个数组能够全部表示出相邻两个块的最大值信息。对于窗口(i, i+k-1)，它的最大值即为max( right[i], left[i+k-1] )。
可以这么理解，若窗口在一个块内，那么right[i]和left[i+k-1]都代表了这个块也就是这个窗口的最大值；若窗口跨越了两个相邻块，
那么right[i]表示左边块部分的最大值，left[i+k-1]表示右边块的最大值，两个中较大的值则代表当前窗口的最大值。



### Python实现

``` python
"""
代码未经优化，看懂就行！！！
"""
from collections import deque
from typing import List

class Solution:

    """
    解法一：暴力求解，双层for循环
    """
    def maxSlidingWindow(self, nums: List[int], k: int) -> List[int]:

        res = []
        for i in range(len(nums) - k + 1):

            cur = nums[i : i+k]
            max = cur[0]
            for j in cur:
                if j > max:
                    max = j
            res.append(max)
        return res

    """
    滑动窗口的经典解法：单调队列
    这里使用双端队列，因为是滑动窗口，所以单调队列保存的是元素的索引，而不是元素本身
    使用前k个元素初始化双端队列，
    滑动，去掉超出窗口长度的元素，去掉小于当前元素的所有值，因为如果比当前值小，必不会是当前窗口的最大值
    单调队列的头部对应的就是最大值
    """
    def maxSlidingWindow2(self, nums: List[int], k: int) -> List[int]:

        res = []
        MonotoneQueue = deque()
        # 初始化双端队列
        for i in range(k):
            if not MonotoneQueue:
                MonotoneQueue.append(i)
            else:
                while MonotoneQueue and nums[i] > nums[MonotoneQueue[-1]]:
                    MonotoneQueue.pop()
                MonotoneQueue.append(i)
        res.append(nums[MonotoneQueue[0]])
        # 从第k个数开始遍历数组
        for i in range(k, len(nums)):
            # 排除超出窗长度的元素
            while (i - MonotoneQueue[0]) > k:
                MonotoneQueue.popleft()
            # 排除所有比当前元素小的值，再插入当前值，维持队列的单调性
            while MonotoneQueue and nums[i] > nums[MonotoneQueue[-1]]:
                MonotoneQueue.pop()
            MonotoneQueue.append(i)
            # 将队头元素汇入结果
            res.append(nums[MonotoneQueue[0]])
        return res


    """
    解法三：动态规划。参考官方题解，那个图很可以理解了，但还是比较难想，典型解法是单调栈或者单调队列
    """
    def maxSlidingWindow3(self, nums: List[int], k: int) -> List[int]:

        if not nums:
            return []
        if k == 1:
            return nums

        # step 1: 将nums按k分块，然后构建每块从左到右、从右到左的最大值
        left, right = [nums[0]]*len(nums), [nums[-1]]*len(nums)     # 用来记录从左到右、从右到左不同块内的最大值
        for i in range(len(nums)):
            if i%k == 0:    # 一个块的开始
                left[i] = nums[i]
            else:
                left[i] = nums[i] if nums[i] > left[i-1] else left[i-1]
        for j in range(len(nums)-2, -1, -1):
            if (j+1)%k == 0:    # 一个块的尾部
                right[j] = nums[j]
            else:
                right[j] = nums[j] if nums[j] > right[j+1] else right[j+1]
        # step 2: 得出结果。left和right两个数组记录了相邻两个块的所有信息
        # 若窗口在块内，从左往右和从右往左正好囊括该块
        # 若窗口跨越两个相邻块，那么从右往左和从左往右可以囊括该块
        res = []
        for i in range(len(nums)-k+1):
            res.append( max(right[i], left[i+k-1]) )
        return res

```
More info: [LeetCode 239](https://leetcode-cn.com/problems/sliding-window-maximum)
