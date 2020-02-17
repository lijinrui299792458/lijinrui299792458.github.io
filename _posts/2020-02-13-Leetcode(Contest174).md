---
layout:     post
title:      Weekly Contest 174
subtitle:   Leetcode竞赛系列
date:       2020-02-13
author:     YunLambert
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - Algorithm

---

> Weekly Contest 174

竞赛链接：[https://leetcode-cn.com/contest/weekly-contest-174/](https://leetcode-cn.com/contest/weekly-contest-174/)

## 方阵中战斗力最弱的 K 行

给你一个大小为 `m * n` 的方阵 `mat`，方阵由若干军人和平民组成，分别用 0 和 1 表示。

请你返回方阵中战斗力最弱的 `k` 行的索引，按从最弱到最强排序。

如果第 ***i*** 行的军人数量少于第 ***j*** 行，或者两行军人数量相同但 ***i*** 小于 ***j***，那么我们认为第 ***i*** 行的战斗力比第 ***j*** 行弱。

军人 **总是** 排在一行中的靠前位置，也就是说 1 总是出现在 0 之前。

**示例 1：**

```
输入：mat = 
[[1,1,0,0,0],
 [1,1,1,1,0],
 [1,0,0,0,0],
 [1,1,0,0,0],
 [1,1,1,1,1]], 
k = 3
输出：[2,0,3]
解释：
每行中的军人数目：
行 0 -> 2 
行 1 -> 4 
行 2 -> 1 
行 3 -> 2 
行 4 -> 5 
从最弱到最强对这些行排序后得到 [2,0,3,1,4]
```

**示例 2：**

```
输入：mat = 
[[1,0,0,0],
 [1,1,1,1],
 [1,0,0,0],
 [1,0,0,0]], 
k = 2
输出：[0,2]
解释： 
每行中的军人数目：
行 0 -> 1 
行 1 -> 4 
行 2 -> 1 
行 3 -> 1 
从最弱到最强对这些行排序后得到 [0,2,3,1]
```

**提示：**

- `m == mat.length`
- `n == mat[i].length`
- `2 <= n, m <= 100`
- `1 <= k <= m`
- `matrix[i][j]` 不是 0 就是 1

解答：

简单题，但是代码不少，尤其是将序号和数量绑定的时候需要进行处理，我用的是map，所以处理起来可能有些麻烦：

```C++
class Solution {
public:
    vector<int> kWeakestRows(vector<vector<int>>& mat, int k) {
        map<int, int> mm;
        vector<int> res;
        int n = mat.size(), m = mat[0].size();
        for (int i = 0; i < n; ++i) {
            mm[i] = 0;
            for (int j = 0; j < m; ++j)
                if (mat[i][j] == 1) mm[i]++;
        }
        
        vector<pair<int, int>> tmp;
        for (auto it = mm.begin(); it != mm.end(); it++) {
            tmp.push_back(make_pair(it->first, it->second));
        }
        sort(tmp.begin(), tmp.end(), [&](pair<int, int> x, pair<int, int> y){
            if(x.second == y.second)
                return x.first < y.first;
            return x.second < y.second;
            });
        
        int i = 0;
        for (auto it = tmp.begin(); i < k && it != tmp.end(); i++, it++) {
            res.push_back(it->first);
        }
        
        return res;
    }
};
```

## 数组大小减半

给你一个整数数组 `arr`。你可以从中选出一个整数集合，并删除这些整数在数组中的每次出现。

返回 **至少** 能删除数组中的一半整数的整数集合的最小大小。

 

**示例 1：**

```
输入：arr = [3,3,3,3,5,5,5,2,2,7]
输出：2
解释：选择 {3,7} 使得结果数组为 [5,5,5,2,2]、长度为 5（原数组长度的一半）。
大小为 2 的可行集合有 {3,5},{3,2},{5,2}。
选择 {2,7} 是不可行的，它的结果数组为 [3,3,3,3,5,5,5]，新数组长度大于原数组的二分之一。
```

**示例 2：**

```
输入：arr = [7,7,7,7,7,7]
输出：1
解释：我们只能选择集合 {7}，结果数组为空。
```

**示例 3：**

```
输入：arr = [1,9]
输出：1
```

**示例 4：**

```
输入：arr = [1000,1000,3,7]
输出：1
```

**示例 5：**

```
输入：arr = [1,2,3,4,5,6,7,8,9,10]
输出：5
```

 

**提示：**

- `1 <= arr.length <= 10^5`
- `arr.length` 为偶数
- `1 <= arr[i] <= 10^5`

解答：

这道题直接用map存储一下每个数出现的次数，如果某几个数出现的次数之和大于总次数的一半，直接输出就可以了：

```C++
class Solution {
public:
    int minSetSize(vector<int>& arr) {
        map<int, int> m;
        int cnt = 0;
        for (int i = 0; i < arr.size(); ++i) m[arr[i]] ++;
        vector<pair<int, int>> tmp; 
        for (auto it = m.begin(); it != m.end(); it++) {
            tmp.push_back(make_pair(it->first, it->second));
        }
        sort(tmp.begin(), tmp.end(), [&](pair<int, int> x, pair<int, int> y){
            if(x.second == y.second)
                return x.first > y.first;
            return x.second > y.second;
            });
        
        int sum = 0;
        for (int i = 0; i< tmp.size(); i++) {
            if (sum >= arr.size() / 2) break;
            sum += tmp[i].second;
            cnt++; 
        }
        return cnt;
    }
};
```

## 分裂树的最大乘积

给你一棵二叉树，它的根为 `root` 。请你删除 1 条边，使二叉树分裂成两棵子树，且它们子树和的乘积尽可能大。

由于答案可能会很大，请你将结果对 10^9 + 7 取模后再返回。

**示例 1：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/02/02/sample_1_1699.png)**

```
输入：root = [1,2,3,4,5,6]
输出：110
解释：删除红色的边，得到 2 棵子树，和分别为 11 和 10 。它们的乘积是 110 （11*10）
```

**示例 2：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/02/02/sample_2_1699.png)

```
输入：root = [1,null,2,3,4,null,null,5,6]
输出：90
解释：移除红色的边，得到 2 棵子树，和分别是 15 和 6 。它们的乘积为 90 （15*6）
```

**示例 3：**

```
输入：root = [2,3,9,10,7,8,6,5,4,11,1]
输出：1025
```

**示例 4：**

```
输入：root = [1,1]
输出：1
```

 

**提示：**

- 每棵树最多有 `50000` 个节点，且至少有 `2` 个节点。
- 每个节点的值在 `[1, 10000]` 之间。

解答：

开始时候想用层序遍历计算遍历到的以该节点为根的子树的和，但是发现TLE了，但是先把第一次提交的代码放在这里：

```C++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    const int m = 1e9+7;
    long getTreeSum(TreeNode* node, long sum) {
        if (node == NULL) return 0;
        return getTreeSum(node->left, sum) + getTreeSum(node->right, sum) + node->val;
    }
    
    int maxProduct(TreeNode* root) {
        long total = getTreeSum(root, 0);
        long res = -1;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            auto x = q.front();
            q.pop();
            long y = getTreeSum(x, 0);
            long tmp = y * (total-y);
            res = max(res, tmp);
            if (x->left) q.push(x->left);
            if (x->right) q.push(x->right);
        }
        return (int)(res%m);
    }
};
```

报TLE的是一棵始终只有右子树的二叉树，这样层序遍历的时候相当于每层只有一个节点，虽然我暂时不知道放入队列和从队列取出的开销有多大，但是肯定比直接存储到数组中要大；所以考虑一次将所有子节点为根的子树的和全部存储下来，减少一点时间开销；还有long和int的存储类型这点需要注意。

```C++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    vector<long> sums;
    static const int mod = 1e9 + 7;
    int maxProduct(TreeNode* root) {
        postOrder(root);
        long res = -1;
        for (int i = 0; i < sums.size() - 1; ++i) {
            // 取最大值时不能取模，应该用long型存结果
            res = max(res, sums[i] * (sums.back() - sums[i]));
        }
        return (int)(res % mod);
    }

    long postOrder(TreeNode* root) {
        if (!root) return 0;
        long res = root->val + postOrder(root->left) + postOrder(root->right);
        sums.push_back(res);
        return res;
    }
};
```

## 跳跃游戏V

给你一个整数数组 `arr` 和一个整数 `d` 。每一步你可以从下标 `i` 跳到：

- `i + x` ，其中 `i + x < arr.length` 且 `0 < x <= d` 。
- `i - x` ，其中 `i - x >= 0` 且 `0 < x <= d` 。

除此以外，你从下标 `i` 跳到下标 `j` 需要满足：`arr[i] > arr[j]` 且 `arr[i] > arr[k]` ，其中下标 `k` 是所有 `i` 到 `j` 之间的数字（更正式的，`min(i, j) < k < max(i, j)`）。

你可以选择数组的任意下标开始跳跃。请你返回你 **最多** 可以访问多少个下标。

请注意，任何时刻你都不能跳到数组的外面。

**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/02/02/meta-chart.jpeg)

```
输入：arr = [6,4,14,6,8,13,9,7,10,6,12], d = 2
输出：4
解释：你可以从下标 10 出发，然后如上图依次经过 10 --> 8 --> 6 --> 7 。
注意，如果你从下标 6 开始，你只能跳到下标 7 处。你不能跳到下标 5 处因为 13 > 9 。你也不能跳到下标 4 处，因为下标 5 在下标 4 和 6 之间且 13 > 9 。
类似的，你不能从下标 3 处跳到下标 2 或者下标 1 处。
```

**示例 2：**

```
输入：arr = [3,3,3,3,3], d = 3
输出：1
解释：你可以从任意下标处开始且你永远无法跳到任何其他坐标。
```

**示例 3：**

```
输入：arr = [7,6,5,4,3,2,1], d = 1
输出：7
解释：从下标 0 处开始，你可以按照数值从大到小，访问所有的下标。
```

**示例 4：**

```
输入：arr = [7,1,7,1,7,1], d = 2
输出：2
```

**示例 5：**

```
输入：arr = [66], d = 1
输出：1
```

 

**提示：**

- `1 <= arr.length <= 1000`
- `1 <= arr[i] <= 10^5`
- `1 <= d <= arr.length`

 解答：

这道题竞赛时没做出来，毕竟看到题面、又感觉是动态规划问题，就已经不想做了。但是看了答案，感觉好像没有想象中的那么难？？？

```C++
class Solution {
public:
	int maxJumps(vector<int>& arr, int d) {
		int n = arr.size();
		vector<vector<int>> temp;
		vector<int> dp(n, 0);
		int res = 1;
		for (int i = 0; i < arr.size(); i++)
			temp.push_back({ arr[i],i });
		sort(temp.begin(), temp.end());

		for (int i = 0; i < n; i++) {
			int index = temp[i][1]; //编号;
			dp[index] = 1;
			//向左找
			for (int j = index - 1; j >= index - d && j >= 0; j--) {
				if (arr[j] >= arr[index]) break;
				if (dp[j] != 0) dp[index] = max(dp[index], dp[j] + 1);
			}
			//向右找
			for (int j = index + 1; j <= index + d && j < n; j++) {
				if (arr[j] >= arr[index]) break;
				if (dp[j] != 0) dp[index] = max(dp[index], dp[j] + 1);
			}
			res = max(dp[index], res);
		}
		return res;
	}
};
```

## 总结

做题速度还是有点慢，以及对动规的天然恐惧......



