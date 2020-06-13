---
layout:     post
title:      Weekly Contest 187
subtitle:   Leetcode竞赛系列
date:       2020-05-03
author:     YunLambert
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - Algorithm

---

> Weekly Contest 187

竞赛链接：[https://leetcode-cn.com/contest/weekly-contest-187/](https://leetcode-cn.com/contest/weekly-contest-187)

## 旅行终点站

给你一份旅游线路图，该线路图中的旅行线路用数组 `paths` 表示，其中 `paths[i] = [cityAi, cityBi]` 表示该线路将会从 `cityAi` 直接前往 `cityBi` 。请你找出这次旅行的终点站，即没有任何可以通往其他城市的线路的城市*。*

题目数据保证线路图会形成一条不存在循环的线路，因此只会有一个旅行终点站。

**示例 1：**

```
输入：paths = [["London","New York"],["New York","Lima"],["Lima","Sao Paulo"]]
输出："Sao Paulo" 
解释：从 "London" 出发，最后抵达终点站 "Sao Paulo" 。本次旅行的路线是 "London" -> "New York" -> "Lima" -> "Sao Paulo" 。
```

**示例 2：**

```
输入：paths = [["B","C"],["D","B"],["C","A"]]
输出："A"
解释：所有可能的线路是：
"D" -> "B" -> "C" -> "A". 
"B" -> "C" -> "A". 
"C" -> "A". 
"A". 
显然，旅行终点站是 "A" 。
```

**示例 3：**

```
输入：paths = [["A","Z"]]
输出："Z"
```

 

**提示：**

- `1 <= paths.length <= 100`
- `paths[i].length == 2`
- `1 <= cityAi.length, cityBi.length <= 10`
- `cityAi != cityBi`
- 所有字符串均由大小写英文字母和空格字符组成。

解答：

用两个数组保存一下起点和终点，寻找在终点数组中没有在起点数组中出现过的站点就ok了。

```C++
class Solution {
public:
    string destCity(vector<vector<string>>& paths) {
        vector<string> s, d;
        for (int i = 0; i < paths.size(); ++i) {
            s.push_back(paths[i][0]);
            d.push_back(paths[i][1]);
        }
        string res;
        for (auto it : d) {
            //cout<<it<<endl;
            auto start = find(s.begin(), s.end(), it);
            if (start == s.end()) {
                res = it;
                break;
            } 
        }
        return res;
    }
};
```

## 是否所有都至少相隔k个元素

给你一个由若干 `0` 和 `1` 组成的数组 `nums` 以及整数 `k`。如果所有 `1` 都至少相隔 `k` 个元素，则返回 `True` ；否则，返回 `False` 。

 

**示例 1：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/05/03/sample_1_1791.png)**

```
输入：nums = [1,0,0,0,1,0,0,1], k = 2
输出：true
解释：每个 1 都至少相隔 2 个元素。
```

**示例 2：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/05/03/sample_2_1791.png)**

```
输入：nums = [1,0,0,1,0,1], k = 2
输出：false
解释：第二个 1 和第三个 1 之间只隔了 1 个元素。
```

**示例 3：**

```
输入：nums = [1,1,1,1,1], k = 0
输出：true
```

**示例 4：**

```
输入：nums = [0,1,0,1], k = 1
输出：true
```

 

**提示：**

- `1 <= nums.length <= 10^5`
- `0 <= k <= nums.length`
- `nums[i]` 的值为 `0` 或 `1`

解答：

暴力直接做，需要其他什么技巧么？

```C++
class Solution {
public:
    bool kLengthApart(vector<int>& nums, int k) {
        int last = -1;
        bool flag = true;
        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] == 1) {
                if (last != -1) {
                    if (i - last - 1 < k) {
                        flag = false;
                        break;
                    }
                }
                last = i;
            }
        }
        return flag;
    }
};
```

## 绝对差不超过限制的最长连续子数组

给你一个整数数组 `nums` ，和一个表示限制的整数 `limit`，请你返回最长连续子数组的长度，该子数组中的任意两个元素之间的绝对差必须小于或者等于 `limit` *。*

如果不存在满足条件的子数组，则返回 `0` 。

 

**示例 1：**

```
输入：nums = [8,2,4,7], limit = 4
输出：2 
解释：所有子数组如下：
[8] 最大绝对差 |8-8| = 0 <= 4.
[8,2] 最大绝对差 |8-2| = 6 > 4. 
[8,2,4] 最大绝对差 |8-2| = 6 > 4.
[8,2,4,7] 最大绝对差 |8-2| = 6 > 4.
[2] 最大绝对差 |2-2| = 0 <= 4.
[2,4] 最大绝对差 |2-4| = 2 <= 4.
[2,4,7] 最大绝对差 |2-7| = 5 > 4.
[4] 最大绝对差 |4-4| = 0 <= 4.
[4,7] 最大绝对差 |4-7| = 3 <= 4.
[7] 最大绝对差 |7-7| = 0 <= 4. 
因此，满足题意的最长子数组的长度为 2 。
```

**示例 2：**

```
输入：nums = [10,1,2,4,7,2], limit = 5
输出：4 
解释：满足题意的最长子数组是 [2,4,7,2]，其最大绝对差 |2-7| = 5 <= 5 。
```

**示例 3：**

```
输入：nums = [4,2,2,2,4,4,2,2], limit = 0
输出：3
```

 

**提示：**

- `1 <= nums.length <= 10^5`
- `1 <= nums[i] <= 10^9`
- `0 <= limit <= 10^9`

解答：

滑动窗口和剪枝来做，竞赛的时候TLE了很多次，滑动窗口没用模板，先放上未AC代码：

```C++
class Solution {
public:
    int longestSubarray(vector<int>& nums, int limit) {
        int start = 0, end = 0;
        int res = 0;
        while (start <= end && end < nums.size()) {
            if (end - start + 1 <= res) {
                end++;
                continue;
            }
            int Min = nums[start];
            int Max = nums[start];
            for (int i = start; i <= end; ++i) {
                Min = min(nums[i], Min);
                Max = max(nums[i], Max);
            }
            if (Max - Min <= limit) {
                res = max(res, end - start + 1);
                end++;
            } else start++;
        }
        return res;
    }
};
```

看了答案后，考虑用了deque来存储最小最大值：

```C++
class Solution {
public:
	int longestSubarray(vector<int> & nums, int limit) {
		deque<int> maxq;
		deque<int> minq;
		int n = nums.size();
		int ans = 1;
		int left = 0;
		for (int i = 0; i < n; ++i) {
			int tmp = nums[i];
			while (!maxq.empty() && nums[maxq.back()] < tmp) {
				maxq.pop_back();
			}
			while (!minq.empty() && nums[minq.back()] > tmp) {
				minq.pop_back();
			}
			if (!maxq.empty() && maxq.front() < left) {
				maxq.pop_front();
			}
			if (!minq.empty() && minq.front() < left) {
				minq.pop_front();
			}
			maxq.push_back(i);
			minq.push_back(i);
			if ((nums[maxq.front()] - nums[minq.front()]) <= limit) {
				ans = max(ans, i - left + 1);
			}
			else {
				++left;
			}
		}
		return ans;
	}
};
```



## 有序矩阵中的第 k 个最小数组和

给你一个 `m * n` 的矩阵 `mat`，以及一个整数 `k` ，矩阵中的每一行都以非递减的顺序排列。

你可以从每一行中选出 1 个元素形成一个数组。返回所有可能数组中的第 k 个 **最小** 数组和。

 

**示例 1：**

```
输入：mat = [[1,3,11],[2,4,6]], k = 5
输出：7
解释：从每一行中选出一个元素，前 k 个和最小的数组分别是：
[1,2], [1,4], [3,2], [3,4], [1,6]。其中第 5 个的和是 7 。  
```

**示例 2：**

```
输入：mat = [[1,3,11],[2,4,6]], k = 9
输出：17
```

**示例 3：**

```
输入：mat = [[1,10,10],[1,4,5],[2,3,6]], k = 7
输出：9
解释：从每一行中选出一个元素，前 k 个和最小的数组分别是：
[1,1,2], [1,1,3], [1,4,2], [1,4,3], [1,1,6], [1,5,2], [1,5,3]。其中第 7 个的和是 9 。 
```

**示例 4：**

```
输入：mat = [[1,1,10],[2,2,9]], k = 7
输出：12
```

 

**提示：**

- `m == mat.length`
- `n == mat.length[i]`
- `1 <= m, n <= 40`
- `1 <= k <= min(200, n ^ m)`
- `1 <= mat[i][j] <= 5000`
- `mat[i]` 是一个非递减数组

解答：

数据量不大，直接暴力做：

```C++
class Solution {
public:
    int kthSmallest(vector<vector<int>>& mat, int k) {
        vector<int> res(mat[0]);
        for (int i = 1; i < mat.size(); ++i) {
            multiset<int> s;
            for (auto x : res) {
                for (auto y : mat[i]) {
                    s.insert(x + y);
                }
            }
            res.assign(s.begin(), s.end());
            if (res.size() > k) res.resize(k);
        }
        return res[k-1];
    }
};
```

还有一个dp解法：[题解](https://leetcode-cn.com/problems/find-the-kth-smallest-sum-of-a-matrix-with-sorted-rows/solution/dpjian-zhi-by-lucifer1004/)

## 总结

感觉这次的所有题不难，暴力解法一般都能想到，但是可能会卡超时。第三题滑动窗口剪枝后还TLE就让人很无语....