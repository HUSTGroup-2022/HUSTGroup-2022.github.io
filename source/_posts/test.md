---
title: 135.分发糖果(Hard)(C++&Java)
date: 2020-10-31 19:06:53
categories:
- LeetCode
- 贪心算法
tags:
- LeetCode
- 贪心算法
---
[LeetCode 135](https://leetcode-cn.com/problems/candy/)，分发糖果问题，难度：`困难`。

## 问题解析

### 问题描述
老师想给孩子们分发糖果，有 N 个孩子站成了一条直线，老师会根据每个孩子的表现，预先给他们评分。
你需要按照以下要求，帮助老师给这些孩子分发糖果：

每个孩子至少分配到 1 个糖果。
相邻的孩子中，评分高的孩子必须获得更多的糖果。
那么这样下来，老师至少需要准备多少颗糖果呢？

More info: [原题](https://leetcode-cn.com/problems/candy/)

### 示例1

>输入:  [1,0,2]
>输出:  5
>解释:  你可以分别给这三个孩子分发 2、1、2 颗糖果。


### 示例2

>输入:  [1,2,2]
>输出:  4
>解释:  你可以分别给这三个孩子分发 1、2、1 颗糖果。第三个孩子只得到 1 颗糖果，这已满足上述两个条件。


### 题解

`贪心算法`采用贪心的策略，保证每次`局部的操作都是当前最优的`，从而使最终一步步累计得到的结果是`全局最优`的。
对于本题，贪心策略通过两次遍历实现：
所有孩子的糖果数初始化为1，`从左往右遍历一遍`，如果右边孩子的评分比左边的高，则右边孩子的糖果数更新为左边孩子的糖果数加1；
再`从右往左遍历一遍`，如果左边孩子的评分比右边的高，且左边孩子当前的糖果数不大于右边孩子的糖果数，则左边孩子的糖果数更新为右边孩子的糖果数加1。
- 通过这两次遍历，即将每个孩子都和左右孩子进行了比较。这里的贪心策略即为，在每次遍历中，只考虑并更新相邻一侧的大小关系。
在样例中，我们初始化糖果分配为`[1,1,1]`，第一次遍历更新后的结果为`[1,1,2]`，第二次遍历更新后的结果为`[2,1,2]`。

### C++实现

``` cpp
int candy(vector<int>& ratings) {
    int size=ratings.size();
    vector<int> num(size,1);
    for(int i=1;i<size;i++)
    {
        if(ratings[i]>ratings[i-1])
        {
            num[i]=num[i-1]+1;
        }
    }
    for(int i=size-1;i>0;i--)
    {
        if(ratings[i-1]>ratings[i])
        {
            if(num[i-1]>num[i]) continue;
            else num[i-1]=num[i]+1;
        }
    }
    return accumulate(num.begin(),num.end(),0);
}
```
More info: [LeetCode 135](https://leetcode-cn.com/problems/candy/)
