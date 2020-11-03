---
title: 218.天际线问题(Hard)(Python)
date: 2020-11-3 10:06:53
categories:
- LeetCode
- 分治算法
- 大顶堆
tags:
- LeetCode
- 分治算法
- 大顶堆
---
[LeetCode 218](https://leetcode-cn.com/problems/the-skyline-problem/)，天际线问题，难度：`困难`。

## 问题解析

### 问题描述
城市中的天际线是从远处望去城市中所有建筑物形成的外部轮廓。假设每个城市为绝对平坦且高度为零的表面
上的完美矩形，以三元组表示（左边界，右边界，高度），且每个建筑物都在合理范围内。

根据给定的三元组，绘制出城市的轮廓线，这些轮廓线输出是关键点的列表，关键点是每个水平线断的左端点。
任何两个相邻建筑物之间的地面也视为天际线轮廓的一部分。具体描述参考leetcode原题。

More info: [原题](https://leetcode-cn.com/problems/the-skyline-problem/)

### 示例1

>输入:  [ [2 9 10], [3 7 15], [5 12 12], [15 20 10], [19 24 8] ]
>输出:  [ [2 10], [3 15], [7 12], [12 0], [15 10], [20 8], [24, 0] ]

### 题解
这题有点难，看了官方题解，“有那么多方法你偏偏选了最难的一种”，以及精选的一种神仙解法，
神仙解法利用堆。不明白为什么能够想出来这样解，这里仅作方法的记录。

解法一：分治算法。把给定的一排建筑物不断的二分切割，最小单元为[]或者一个建筑物。对于一个
建筑物来说，天际线即为（左端点，高度）（右端点，零）。分割之后再归并，归并的步骤如下：
left ==> 左天际线轮廓， right ==> 右天际线轮廓
ileft ==> 左天际线指针， iright ==> 右天际线指针
lHeight ==>左天际线当前高度， rHeight ==> 右天际线当前高度
结果数组初始化为空
if left[ileft] < right[iright]: 说明左天际线在右天际线的左边，考虑更新左边
    天际线点 = [ left[ileft], max( left[ileft].height, rHeight )
    lHeight = left[ileft].height    # 更新当前高度
    ileft向前移动一位                  # 更新指针
elseif left[ileft] > right[iright]: 说明右天际线在左天际线左边，考虑更新右边
    原理同上
else: 左天际线和右天际线相同，这时候高度高的会遮住高度低的
    天际线点 = [left[ileft], max( left[ileft].height, right[iright].height )
    更新lHeight, rHeight
    更新ileft, iright
如果结果数组为空或者当前天际线点的高度和结果中最后一个点的高度不同：
    这样才把当前天际线点放入结果数组中
如果左右某一个天际线轮廓没有处理完，则直接添加到结果数组中


解法二：神仙的线扫描算法。大体思想如下：把每个建筑物拆分为水平线段两端两个点，即（左端点，高度）
（右端点，高度），同时，把左端点的高度变为负的，这样是为了给所有的端点排序时不会出错，也就是两个
建筑物拥有同一个左边界时侯特殊情况的处理。堆初始化含0元素。初始化最大高度为0。
对排序之后的所有端点遍历：
如果遇到左端点，恢复高度并入堆，如果是右端点，则从堆中除掉这个高度。
从堆中弹出最大元素，为当前的最大高度，如果最大高度改变了，说明发生了转折，则记录下当前天际线点为
（端点，最大高度）并汇入结果中。

多说两句，分治算法可能是比较好想，但归并的过程确实是比较复杂难想。解法二就想不到了，解法二中的
高度变负技巧、堆中去掉一个元素、堆初始化为0，比较高级。

### Python实现

``` python
from typing import List

class Solution:

    """
    解法一：divide and conquer
    """
    def getSkyline(self, buildings: List[List[int]]) -> List[List[int]]:

        if len(buildings) == 0:
            return []
        if len(buildings) == 1:
            return [ [buildings[0][0], buildings[0][2]], [buildings[0][1], 0] ]    # 左天际线， 右天际线

        mid = len(buildings) // 2
        left = self.getSkyline(buildings[:mid])
        right = self.getSkyline(buildings[mid:])

        return self.__merge(left, right)

    def __merge(self, left, right):

        hleft, hright = 0, 0     # 记录当前的高度
        ileft, iright = 0, 0     # 左右两边的指针
        res = []
        while ileft < len(left) and iright < len(right):

            # 如果左边的x坐标小于右边的x坐标，那么更新左边的
            if left[ileft][0] < right[iright][0]:
                # 当前天际线
                cp = [left[ileft][0], max(left[ileft][1], hright)]
                # 当前高度
                hleft = left[ileft][1]
                ileft += 1
            # 如果左边的x坐标大于右边的x左边，那么更新右边的
            elif left[ileft][0] > right[iright][0]:
                # 当前天际线
                cp = [right[iright][0], max(hleft, right[iright][1])]
                hright = right[iright][1]
                iright += 1
            # 如果左右两边的x相等，那么高度较小的会被遮挡住，左右同时更新
            else:
                cp = [left[ileft][0], max(left[ileft][1], right[iright][1])]
                hleft, hright = left[ileft][1], right[iright][1]
                ileft += 1
                iright += 1

            # 当前最大高度不等于之前的最大高度时，才将当前的cp放入结果
            if len(res) == 0 or cp[1] != res[-1][1]:
                res.append(cp)
        res.extend(left[ileft:] or right[iright:])    # 此处不能用append方法，因为left或者right本身是一个二维数组
        return res


    """
    解法二：神仙解法，利用堆，线扫描，小技巧
    """
    def getSkyline2(self, buildings: List[List[int]]) -> List[List[int]]:
        # step 1: 将每个building分成左端点和右端点，即高度横线的两个端点
        all = []
        for build in buildings:
            all.append([build[0], -build[2]])    # 左端点， 负高度以排序
            all.append([build[1], build[2]])       # 右端点， 正高度
        # step 2： 给all排序，这样会根据x排序，如果x相同，左端点因为是负值会排在右端点前面，
        all.sort()

        # step 3: 开始扫描，如果是左端点，入大顶堆，如果是右端点，从堆中去掉该点对应的高度
        # 从堆中去掉一个元素目前还没有实现，因此以复杂度换简单
        # 大顶堆记录的是当前高度以及最大高度
        # 如果最大高度变了，说明出现了转折点，那么填入结果
        res = []
        hheap = [0]
        last = [0, 0]     # 记录上一个时刻的天际线点
        for p in all:
            if p[1] < 0:    # 左端点，高度入堆
                hheap.append(-p[1])
            else:           # 右端点，高度除堆
                hheap.remove(p[1])

            # 大顶堆弹出最大元素，为当前最大高度
            maxheight = max(hheap)

            # 如果当前高度不同于前一高度，则说明发生了转折
            if maxheight != last[1]:
                last[0] = p[0]
                last[1] = maxheight
                res.append([p[0],maxheight])    # 此处如果直接append(last)存入的是对last的引用，如果last改变，那么res中的内容也会改变

        return res

```
More info: [LeetCode 218](https://leetcode-cn.com/problems/the-skyline-problem/)
