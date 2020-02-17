---
layout:     post
title:      Weekly Contest 176
subtitle:   Leetcode竞赛系列
date:       2020-02-16
author:     YunLambert
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - Algorithm

---

> Weekly Contest 176

竞赛链接：[https://leetcode-cn.com/contest/weekly-contest-176/](https://leetcode-cn.com/contest/weekly-contest-176)

## 统计有序矩阵中的负数

给你一个 m * n 的矩阵 grid，矩阵中的元素无论是按行还是按列，都以非递增顺序排列。 

请你统计并返回 grid 中 负数 的数目。

示例 1：

输入：grid = [[4,3,2,-1],[3,2,1,-1],[1,1,-1,-2],[-1,-1,-2,-3]]
输出：8
解释：矩阵中共有 8 个负数。
示例 2：

输入：grid = [[3,2],[1,0]]
输出：0
示例 3：

输入：grid = [[1,-1],[-1,-1]]
输出：3
示例 4：

输入：grid = [[-1]]
输出：1


提示：

m == grid.length
n == grid[i].length
1 <= m, n <= 100
-100 <= grid[i][j] <= 100

解答：

暴力，简单。

```C++
class Solution {
public:
    int countNegatives(vector<vector<int>>& grid) {
        int n = grid.size(), m = grid[0].size();
        int cnt = 0;
        for (int i = 0; i < n; ++i) {
            for (int j = m -1; j >= 0; j--) {
                if (grid[i][j] >= 0) break;
                cnt++;
            }
        }
        return cnt;
    }
};
```

## 最后k个数的乘积

请你实现一个「数字乘积类」ProductOfNumbers，要求支持下述两种方法：

1. add(int num)

将数字 num 添加到当前数字列表的最后面。
2. getProduct(int k)

返回当前数字列表中，最后 k 个数字的乘积。
你可以假设当前列表中始终 至少 包含 k 个数字。
题目数据保证：任何时候，任一连续数字序列的乘积都在 32-bit 整数范围内，不会溢出。

示例：
```
输入：
["ProductOfNumbers","add","add","add","add","add","getProduct","getProduct","getProduct","add","getProduct"]
[[],[3],[0],[2],[5],[4],[2],[3],[4],[8],[2]]

输出：
[null,null,null,null,null,null,20,40,0,null,32]

解释：
ProductOfNumbers productOfNumbers = new ProductOfNumbers();
productOfNumbers.add(3);        // [3]
productOfNumbers.add(0);        // [3,0]
productOfNumbers.add(2);        // [3,0,2]
productOfNumbers.add(5);        // [3,0,2,5]
productOfNumbers.add(4);        // [3,0,2,5,4]
productOfNumbers.getProduct(2); // 返回 20 。最后 2 个数字的乘积是 5 * 4 = 20
productOfNumbers.getProduct(3); // 返回 40 。最后 3 个数字的乘积是 2 * 5 * 4 = 40
productOfNumbers.getProduct(4); // 返回  0 。最后 4 个数字的乘积是 0 * 2 * 5 * 4 = 0
productOfNumbers.add(8);        // [3,0,2,5,4,8]
productOfNumbers.getProduct(2); // 返回 32 。最后 2 个数字的乘积是 4 * 8 = 32 

```


提示：

add 和 getProduct 两种操作加起来总共不会超过 40000 次。
0 <= num <= 100
1 <= k <= 40000

解答：

由于LC的数据较弱，暴力做可以,但是实际上可以考虑前缀积这个做法，但是需要注意在加入数组时记录0的位置并在计算乘积时把0替换成1：

```C++
class ProductOfNumbers {
    public:
    int mult[40010],isZeroPos,n;//前缀连乘积(且把0视为1)
    ProductOfNumbers() {
        n = 0;//当前添加的数的下标
        isZeroPos = 0;//若包含该下标 则连乘积为0
        mult[0] = 1;
    }
    void add(int num) {
        ++n;
        if(num == 0) 
            isZeroPos = n, mult[n] = 1;//更新0点,因为该点对后面的数的乘积无影响故令此处连乘积为 1
        else mult[n] = mult[n-1]*num; 
    }
    int getProduct(int k) {
        k = n-k+1;//也就是从前往后数的第n-k+1个数
        if(k <= isZeroPos) return 0;//包含了0 所以连乘积一定为0
        else return mult[n]/mult[k-1];//若不包含0 则为k~n的连乘积=1~n的连乘积/1~k-1的连乘积
    }
};
```

## 最多可以参加的会议

给你一个数组 `events`，其中 `events[i] = [startDayi, endDayi]` ，表示会议 `i` 开始于 `startDayi` ，结束于 `endDayi` 。

你可以在满足 `startDayi <= d <= endDayi` 中的任意一天 `d` 参加会议 `i` 。注意，一天只能参加一个会议。

请你返回你可以参加的 **最大** 会议数目。

**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/02/16/e1.png)

```
输入：events = [[1,2],[2,3],[3,4]]
输出：3
解释：你可以参加所有的三个会议。
安排会议的一种方案如上图。
第 1 天参加第一个会议。
第 2 天参加第二个会议。
第 3 天参加第三个会议。
```

**示例 2：**

```
输入：events= [[1,2],[2,3],[3,4],[1,2]]
输出：4
```

**示例 3：**

```
输入：events = [[1,4],[4,4],[2,2],[3,4],[1,1]]
输出：4
```

**示例 4：**

```
输入：events = [[1,100000]]
输出：1
```

**示例 5：**

```
输入：events = [[1,1],[1,2],[1,3],[1,4],[1,5],[1,6],[1,7]]
输出：7
```

 

**提示：**

- `1 <= events.length <= 10^5`
- `events[i].length == 2`
- `1 <= events[i][0] <= events[i][1] <= 10^5`

解答：

以下是摘自leetcode题解区的一篇回答：

一道典型的扫描算法题。由于每个时间点最多参加一个会议，我们可以从1开始遍历所有时间。

对于每一个时间点，所有在当前时间及之前时间开始，并且在当前时间还未结束的会议都是可参加的。显然，在所有可参加的会议中，选择结束时间最早的会议是最优的，因为其他会议还有更多的机会可以去参加。

怎样动态获得当前结束时间最早的会议呢？我们可以使用一个小根堆记录所有当前可参加会议的结束时间。在每一个时间点，我们首先将当前时间点开始的会议加入小根堆，再把当前已经结束的会议移除出小根堆（因为已经无法参加了），然后从剩下的会议中选择一个结束时间最早的去参加。

为了快速获得当前时间点开始的会议，我们以O(N)时间预处理得到每个时间点开始的会议的序号。

算法总的时间复杂度为$O(Tlog N)$（这里的T为时间范围）

```C++
const int MAX = 1e5 + 1;

class Solution {
public:
    int maxEvents(vector<vector<int>>& events) {
        vector<vector<int>> left(MAX);
        for (int i = 0; i < events.size(); ++i)
            left[events[i][0]].emplace_back(i);
        
        int ans = 0;
        priority_queue<int, vector<int>, greater<>> pq;
        for (int i = 1; i < MAX; ++i) {
            for (int j : left[i])
                pq.push(events[j][1]);
            while (!pq.empty() && pq.top() < i)
                pq.pop();
            if (!pq.empty()) {
                pq.pop();
                ans++;
            }
        }
        return ans;
    }
};
```

## 多次求和构造目标数组

给你一个整数数组 `target` 。一开始，你有一个数组 `A` ，它的所有元素均为 1 ，你可以执行以下操作：

- 令 `x` 为你数组里所有元素的和
- 选择满足 `0 <= i < target.size` 的任意下标 `i` ，并让 `A` 数组里下标为 `i` 处的值为 `x` 。
- 你可以重复该过程任意次

如果能从 `A` 开始构造出目标数组 `target` ，请你返回 True ，否则返回 False 。

 

**示例 1：**

```
输入：target = [9,3,5]
输出：true
解释：从 [1, 1, 1] 开始
[1, 1, 1], 和为 3 ，选择下标 1
[1, 3, 1], 和为 5， 选择下标 2
[1, 3, 5], 和为 9， 选择下标 0
[9, 3, 5] 完成
```

**示例 2：**

```
输入：target = [1,1,1,2]
输出：false
解释：不可能从 [1,1,1,1] 出发构造目标数组。
```

**示例 3：**

```
输入：target = [8,5]
输出：true
```

 

**提示：**

- `N == target.length`
- `1 <= target.length <= 5 * 10^4`
- `1 <= target[i] <= 10^9`

解答：

```C++
class Solution {
public:
    bool flag = false;
    
    void dfs(vector<int>& target, long sum) {
        if (flag) return;
        if (sum == target.size()) {flag = true;return;}
        for (int i = 0; i < target.size(); i++) {
            if ((long)2*target[i] <= sum) continue;
            int origin = target[i];
            target[i] = 2*target[i] - sum;
            dfs(target, sum-origin+target[i]);
            target[i] = origin;
        }
    }
    
    bool isPossible(vector<int>& target) {
        long sum = 0;
        for (auto num: target) {
            if (sum >= (long)2*INT_MAX-num) {return false;}
            sum += num;
        }
        dfs(target, sum);
        
        return flag;
    }
};
```



## 总结

血崩的一次，前两道题靠暴力莽过了，LC的数据有点弱....... 第三题以为是以前做过的贪心问题，结果发现理解错了题意；第四题看答案是找规律，可我当时第一个想法是dfs，并且边界没考虑清楚....



