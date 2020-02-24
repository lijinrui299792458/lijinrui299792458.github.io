---
layout:     post
title:      Weekly Contest 177
subtitle:   Leetcode竞赛系列
date:       2020-02-24
author:     YunLambert
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - Algorithm

---

> Weekly Contest 177

竞赛链接：[https://leetcode-cn.com/contest/weekly-contest-177/](https://leetcode-cn.com/contest/weekly-contest-177)

## 日期之间隔几天

请你编写一个程序来计算两个日期之间隔了多少天。

日期以字符串形式给出，格式为 `YYYY-MM-DD`，如示例所示。

 

**示例 1：**

```
输入：date1 = "2019-06-29", date2 = "2019-06-30"
输出：1
```

**示例 2：**

```
输入：date1 = "2020-01-15", date2 = "2019-12-31"
输出：15
```

 

**提示：**

- 给定的日期是 `1971` 年到 `2100` 年之间的有效日期。

解答：

从1970-0-0开始算到日期，然后再相减一下。

```C++
class Solution {
public:
    bool isYear(int y) {
        return ((y%4 == 0 && y % 100) || y % 400 == 0);
    }
    
    int getDay(string s) {
        int days[13] = {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
        int y = 0, m = 0, d = 0;
        for (int i = 0; i <= 3; ++i) y = y*10 + s[i] - '0';
        for (int i = 5; i <= 6; ++i) m = m*10 + s[i] - '0';
        for (int i = 8; i <= 9; ++i) d = d*10 + s[i] - '0';
        int res = 0;
        for (int i = 1970; i <= y - 1; ++i) {
            if (isYear(i)) res += 366;
            else res += 365;
        }
        if (isYear(y)) days[2] = 29;
        for (int i = 1; i <= m - 1; ++i) res += days[i]; 
        res += d;
        return res;
    }
    
    int daysBetweenDates(string date1, string date2) {
        return abs(getDay(date1) - getDay(date2));
    }
};
```



## 验证二叉树

二叉树上有 `n` 个节点，按从 `0` 到 `n - 1` 编号，其中节点 `i` 的两个子节点分别是 `leftChild[i]` 和 `rightChild[i]`。

只有 **所有** 节点能够形成且 **只** 形成 **一颗** 有效的二叉树时，返回 `true`；否则返回 `false`。

如果节点 `i` 没有左子节点，那么 `leftChild[i]` 就等于 `-1`。右子节点也符合该规则。

注意：节点没有值，本问题中仅仅使用节点编号。

 

**示例 1：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/02/23/1503_ex1.png)**

```
输入：n = 4, leftChild = [1,-1,3,-1], rightChild = [2,-1,-1,-1]
输出：true
```

**示例 2：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/02/23/1503_ex2.png)**

```
输入：n = 4, leftChild = [1,-1,3,-1], rightChild = [2,3,-1,-1]
输出：false
```

**示例 3：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/02/23/1503_ex3.png)**

```
输入：n = 2, leftChild = [1,0], rightChild = [-1,-1]
输出：false
```

**示例 4：**

**![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/02/23/1503_ex4.png)**

```
输入：n = 6, leftChild = [1,-1,-1,4,-1,-1], rightChild = [2,-1,-1,5,-1,-1]
输出：false
```

 

**提示：**

- `1 <= n <= 10^4`
- `leftChild.length == rightChild.length == n`
- `-1 <= leftChild[i], rightChild[i] <= n - 1`

解答：

首先觉得是并查集的思想，最后判断一下连通块的数量；如果在过程中发现目标节点已经提前在连通块中，说明会出现示例2的呢种情况，所以提前return false。

```C++
class Solution {
public:
    int find(int x) {
        int r = x;
        while (pre[r] != r) r = pre[r];
        int i = x, j;
        while (pre[i] != r) {
            j = pre[i];
            pre[i] = r;
            i = j;
        }
        return r;
    }

    bool join(int x, int y) {
        int fx = find(x), fy = find(y);
        if (fx == fy) return false;
        if (fx > fy) swap(fx, fy);
        pre[fx] = fy;
        return true; 
    }

    bool validateBinaryTreeNodes(int n, vector<int>& leftChild, vector<int>& rightChild) {
        for (int i = 0; i < n; ++i) pre[i] = i;
        int cnt = n;
        for (int i = 0; i < n; ++i) {
            if (leftChild[i] != -1) {
                if (!join(i, leftChild[i])) return false;
                else --cnt;
            }
            if (rightChild[i] != -1) {
                if (!join(i, rightChild[i])) return false;
                else --cnt;
            }
        }
        return cnt == 1;
    }
private:
    int pre[10010];
};
```

其实还有另一种解法 ,判断一下父节点的数量

```C++
class Solution {
public:
    bool validateBinaryTreeNodes(int n, vector<int>& leftChild, vector<int>& rightChild) {
        vector<int> par[20010];
        for (int i = 0; i < n; ++i) {
            if (leftChild[i] != -1) par[leftChild[i]].push_back(i);
            if (rightChild[i] != -1) par[rightChild[i]].push_back(i);
        }
        int root = 0, nodes = 0;
        for (int i = 0; i < n; ++i) {
            if (par[i].size() == 2) return false;
            if (par[i].size() == 0) root++;
            else nodes++;
        }
        if (root == 1 && nodes == n - 1) return true;
        return false;
    }
};
```

## 最接近的因数

给你一个整数 `num`，请你找出同时满足下面全部要求的两个整数：

- 两数乘积等于  `num + 1` 或 `num + 2`
- 以绝对差进行度量，两数大小最接近

你可以按任意顺序返回这两个整数。

 

**示例 1：**

```
输入：num = 8
输出：[3,3]
解释：对于 num + 1 = 9，最接近的两个因数是 3 & 3；对于 num + 2 = 10, 最接近的两个因数是 2 & 5，因此返回 3 & 3 。
```

**示例 2：**

```
输入：num = 123
输出：[5,25]
```

**示例 3：**

```
输入：num = 999
输出：[40,25]
```

 

**提示：**

- `1 <= num <= 10^9`

解答：

注意判断范围就可以：

```C++
class Solution {
public:
    typedef pair<int, int> PII;
    PII disint(int x) {
        PII res;
        int d = 1e9+10;
        for (int i = 1; i <= sqrt(x); ++i) {
            if (x % i == 0) {
                if (abs(i - x / i) < d) {
                    d = abs(i - x/i);
                    res = {i, x/i};
                }
            }
        }
        return res;
    }
    vector<int> closestDivisors(int num) {
        vector<int> res;
        PII a = disint(num + 1);
        PII b = disint(num + 2);
        if (abs(a.first - a.second) < abs(b.first - b.second)) {
            res.push_back(a.first);
            res.push_back(a.second);
        }
        else {
            res.push_back(b.first);
            res.push_back(b.second);
        }
        return res;
    }
};
```

## 形成三的最大倍数

给你一个整数数组 digits，你可以通过按任意顺序连接其中某些数字来形成 3 的倍数，请你返回所能得到的最大的 3 的倍数。

由于答案可能不在整数数据类型范围内，请以字符串形式返回答案。

如果无法得到答案，请返回一个空字符串。

示例 1：

输入：digits = [8,1,9]
输出："981"
示例 2：

输入：digits = [8,6,7,1,0]
输出："8760"
示例 3：

输入：digits = [1]
输出：""
示例 4：

输入：digits = [0,0,0,0,0,0]
输出："0"


提示：

1 <= digits.length <= 10^4
0 <= digits[i] <= 9
返回的结果不应包含不必要的前导零。

解答：

直接去看了[答案](https://leetcode-cn.com/problems/largest-multiple-of-three/solution/c-qu-diao-zui-xiao-zhi-8ms-by-yusenzhang_chatc/)：如果取模3等于0，那其实可以都要，如果是1，那就得去掉一个1或者两个2，如果是2那就得去掉一个2或者两个1.删掉一个数的函数其实是类似的，可以反复调用。

```C++
class Solution {
    int cnt[10],sum;
    string ans = "";
    set<int> s;
    int del(int m)
    {
        for(int i=m;i<=9;i+=3)if(cnt[i]){cnt[i]--;return 1;}
        return 0;
    }
public:
    string largestMultipleOfThree(vector<int>& d) {
        for(auto x:d)cnt[x]++,sum+=x;
        if(sum%3==1)if(!del(1))del(2),del(2);
        if(sum%3==2)if(!del(2))del(1),del(1);
        for(int i=9;i>=0;i--)while(cnt[i]--)ans+=i+'0',s.insert(i);
        if(s.size()==1 && s.count(0))return "0";
        return ans;
    }
};
```

