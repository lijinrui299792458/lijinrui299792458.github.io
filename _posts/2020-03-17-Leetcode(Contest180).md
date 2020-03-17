---
layout:     post
title:      Weekly Contest 180
subtitle:   Leetcode竞赛系列
date:       2020-03-17
author:     YunLambert
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - Algorithm

---

> Weekly Contest 180

竞赛链接：[https://leetcode-cn.com/contest/weekly-contest-180/](https://leetcode-cn.com/contest/weekly-contest-180)

## 矩阵中的幸运数

给你一个 `m * n` 的矩阵，矩阵中的数字 **各不相同** 。请你按 **任意** 顺序返回矩阵中的所有幸运数。

幸运数是指矩阵中满足同时下列两个条件的元素：

- 在同一行的所有元素中最小
- 在同一列的所有元素中最大

 

**示例 1：**

```
输入：matrix = [[3,7,8],[9,11,13],[15,16,17]]
输出：[15]
解释：15 是唯一的幸运数，因为它是其所在行中的最小值，也是所在列中的最大值。
```

**示例 2：**

```
输入：matrix = [[1,10,4,2],[9,3,8,7],[15,16,17,12]]
输出：[12]
解释：12 是唯一的幸运数，因为它是其所在行中的最小值，也是所在列中的最大值。
```

**示例 3：**

```
输入：matrix = [[7,8],[1,2]]
输出：[7]
```

 

**提示：**

- `m == mat.length`
- `n == mat[i].length`
- `1 <= n, m <= 50`
- `1 <= matrix[i][j] <= 10^5`
- 矩阵中的所有元素都是不同的

解答：

签到题，很简单:

````C++
class Solution {
public:
    vector<int> luckyNumbers (vector<vector<int>>& matrix) {
        int n = matrix.size();
        if (n < 1) return res;
        int m = matrix[0].size();

        for (int i = 0; i < n; ++i) {
            int f_min = INT_MAX;
            for (int j = 0; j < m; ++j) {
                f_min = f_min > matrix[i][j]? matrix[i][j] : f_min;
            }
            t1.push_back(f_min);
        }
        
        for (int i = 0; i < m; ++i) {
            int f_max = INT_MIN;
            for (int j = 0; j < n; ++j) {
                f_max = f_max < matrix[j][i]? matrix[j][i] : f_max;
            }
            t2.push_back(f_max);
        }
        
        for (auto x1 : t1) {
            for (auto x2 : t2) {
                if (x1 == x2) res.push_back(x1);
            }
        }
        return res;
    }
private:
    vector<int> t1, t2;
    vector<int> res;
};
````



## 设计一个支持增量操作的栈

请你设计一个支持下述操作的栈。

实现自定义栈类 `CustomStack` ：

- `CustomStack(int maxSize)`：用 `maxSize` 初始化对象，`maxSize` 是栈中最多能容纳的元素数量，栈在增长到 `maxSize` 之后则不支持 `push` 操作。
- `void push(int x)`：如果栈还未增长到 `maxSize` ，就将 `x` 添加到栈顶。
- `int pop()`：返回栈顶的值，或栈为空时返回 **-1** 。
- `void inc(int k, int val)`：栈底的 `k` 个元素的值都增加 `val` 。如果栈中元素总数小于 `k` ，则栈中的所有元素都增加 `val` 。

 

**示例：**

```
输入：
["CustomStack","push","push","pop","push","push","push","increment","increment","pop","pop","pop","pop"]
[[3],[1],[2],[],[2],[3],[4],[5,100],[2,100],[],[],[],[]]
输出：
[null,null,null,2,null,null,null,null,null,103,202,201,-1]
解释：
CustomStack customStack = new CustomStack(3); // 栈是空的 []
customStack.push(1);                          // 栈变为 [1]
customStack.push(2);                          // 栈变为 [1, 2]
customStack.pop();                            // 返回 2 --> 返回栈顶值 2，栈变为 [1]
customStack.push(2);                          // 栈变为 [1, 2]
customStack.push(3);                          // 栈变为 [1, 2, 3]
customStack.push(4);                          // 栈仍然是 [1, 2, 3]，不能添加其他元素使栈大小变为 4
customStack.increment(5, 100);                // 栈变为 [101, 102, 103]
customStack.increment(2, 100);                // 栈变为 [201, 202, 103]
customStack.pop();                            // 返回 103 --> 返回栈顶值 103，栈变为 [201, 202]
customStack.pop();                            // 返回 202 --> 返回栈顶值 202，栈变为 [201]
customStack.pop();                            // 返回 201 --> 返回栈顶值 201，栈变为 []
customStack.pop();                            // 返回 -1 --> 栈为空，返回 -1
```

 

**提示：**

- `1 <= maxSize <= 1000`
- `1 <= x <= 1000`
- `1 <= k <= 1000`
- `0 <= val <= 100`
- 每种方法 `increment`，`push` 以及 `pop` 分别最多调用 `1000` 次

解答：

数据量不大，直接暴力就可以做：

```C++
class CustomStack {
public:
    CustomStack(int maxSize) {
        Size = maxSize;
    }
    
    void push(int x) {
        if (s1.size() < Size) s1.push(x);
    }
    
    int pop() {
        if (s1.empty()) return -1;
        int x = s1.top();
        s1.pop();
        return x;
    }
    
    void increment(int k, int val) {
        int len;
        if (s1.size() <= k) len = 0;
        else len = s1.size() - k;
        
        while (!s1.empty()) {
            int x = s1.top();
            s1.pop();
            if (len == 0) {
                s2.push(x + val);
            } else {
                s2.push(x);
                len--;
            }
        }
        
        while (!s2.empty()) {
            int x = s2.top();
            s1.push(x);
            s2.pop();
        }
    }
private:
    stack<int> s1, s2;
    int Size;
};

/**
 * Your CustomStack object will be instantiated and called as such:
 * CustomStack* obj = new CustomStack(maxSize);
 * obj->push(x);
 * int param_2 = obj->pop();
 * obj->increment(k,val);
 */
```

## 将二叉搜索树变平衡

给你一棵二叉搜索树，请你返回一棵 **平衡后** 的二叉搜索树，新生成的树应该与原来的树有着相同的节点值。

如果一棵二叉搜索树中，每个节点的两棵子树高度差不超过 1 ，我们就称这棵二叉搜索树是 **平衡的** 。

如果有多种构造方法，请你返回任意一种。

 

**示例：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/03/15/1515_ex1.png)![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/03/15/1515_ex1_out.png)**

```
输入：root = [1,null,2,null,3,null,4,null,null]
输出：[2,1,3,null,null,null,4]
解释：这不是唯一的正确答案，[3,1,4,null,2,null,null] 也是一个可行的构造方案。
```

 

**提示：**

- 树节点的数目在 `1` 到 `10^4` 之间。
- 树节点的值互不相同，且在 `1` 到 `10^5` 之间。

解答：

前序遍历后选取中间的节点进行重建树：

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
const int MAXN = 1e4+50;
int vec[MAXN], top;

void dfs(TreeNode* node) {
    if (!node) return;
    dfs(node->left);
    vec[++top] = node->val;
    dfs(node->right);
}

TreeNode* rebuild(int left, int right) {
    if (left > right) return NULL;
    int mid =(left + right)/2;
    TreeNode* ret = new TreeNode(vec[mid]);
    ret->left = rebuild(left, mid-1);
    ret->right = rebuild(mid+1, right);
    return ret;
}
class Solution {
public:
    TreeNode* balanceBST(TreeNode* root) {
        top = 0;
        dfs(root);
        return rebuild(1, top);
    }
};
```

## 最大团队表现值

公司有编号为 `1` 到 `n` 的 `n` 个工程师，给你两个数组 `speed` 和 `efficiency` ，其中 `speed[i]` 和 `efficiency[i]` 分别代表第 `i` 位工程师的速度和效率。请你返回由最多 `k` 个工程师组成的 **最大团队表现值** ，由于答案可能很大，请你返回结果对 `10^9 + 7` 取余后的结果。

**团队表现值** 的定义为：一个团队中「所有工程师速度的和」乘以他们「效率值中的最小值」。

 

**示例 1：**

```
输入：n = 6, speed = [2,10,3,1,5,8], efficiency = [5,4,3,9,7,2], k = 2
输出：60
解释：
我们选择工程师 2（speed=10 且 efficiency=4）和工程师 5（speed=5 且 efficiency=7）。他们的团队表现值为 performance = (10 + 5) * min(4, 7) = 60 。
```

**示例 2：**

```
输入：n = 6, speed = [2,10,3,1,5,8], efficiency = [5,4,3,9,7,2], k = 3
输出：68
解释：
此示例与第一个示例相同，除了 k = 3 。我们可以选择工程师 1 ，工程师 2 和工程师 5 得到最大的团队表现值。表现值为 performance = (2 + 10 + 5) * min(5, 4, 7) = 68 。
```

**示例 3：**

```
输入：n = 6, speed = [2,10,3,1,5,8], efficiency = [5,4,3,9,7,2], k = 4
输出：72
```

 

**提示：**

- `1 <= n <= 10^5`
- `speed.length == n`
- `efficiency.length == n`
- `1 <= speed[i] <= 10^5`
- `1 <= efficiency[i] <= 10^8`
- `1 <= k <= n`

解答：

枚举+优先队列：

```C++
const int MAXN = 1e5+10;
struct engineer {
    long s, e;
}p[MAXN];
const long long MOD = 1e9+7;
inline bool cmp(const engineer& a, const engineer& b) {
    return a.e>b.e;
}

class Solution {
public:
    int maxPerformance(int n, vector<int>& speed, vector<int>& efficiency, int k) {
        for (int i = 0; i < speed.size(); ++i) {
            p[i].s = speed[i];
            p[i].e = efficiency[i];
        }
        sort(p, p+n, cmp);
        
        while (!que.empty()) que.pop();
        
        long long sum = 0;
        long long res = 0;
        for (int i = 0; i < n; ++i) {
            sum += p[i].s;
            que.push(-p[i].s);
        
            while (que.size() > k) {
                sum += que.top();
                que.pop();
            }
            res = max(res, sum*p[i].e);
       }
        return res%MOD;
    }
private:
    priority_queue<long long> que;
}; 
```

## 总结

整体不难，没有dp就是万幸。

