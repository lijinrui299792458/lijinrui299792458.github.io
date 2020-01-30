---
layout:     post
title:      Weekly Contest 171
subtitle:   Leetcode竞赛系列
date:       2020-01-29
author:     YunLambert
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - Algorithm

---

> Weekly Contest 171

竞赛链接：[https://leetcode-cn.com/contest/weekly-contest-171/](https://leetcode-cn.com/contest/weekly-contest-171/)

## 1.将整数转换为两个无零整数的和

「无零整数」是十进制表示中 不含任何 0 的正整数。

给你一个整数 n，请你返回一个 由两个整数组成的列表 [A, B]，满足：

A 和 B 都是无零整数
A + B = n
题目数据保证至少有一个有效的解决方案。

如果存在多个有效解决方案，你可以返回其中任意一个。

 示例 1：

输入：n = 2
输出：[1,1]
解释：A = 1, B = 1. A + B = n 并且 A 和 B 的十进制表示形式都不包含任何 0 。
示例 2：

输入：n = 11
输出：[2,9]
示例 3：

输入：n = 10000
输出：[1,9999]
示例 4：

输入：n = 69
输出：[1,68]
示例 5：

输入：n = 1010
输出：[11,999]


提示：

2 <= n <= 10^4

### 解答

签到题，判断两个数是否包含0就行了：

```C++
class Solution {
public:
    bool isContainsZero(int n) {
        while (n) {
            if (n%10 == 0) return true;
            n /= 10;
        }
        return false;
    }
    vector<int> getNoZeroIntegers(int n) {
        vector<int> res;
        for (int i = 1; i <= n/2; ++i) {
            if (!isContainsZero(i) && !isContainsZero(n-i)) {
                res.push_back(i);
                res.push_back(n-i);
                break;
            }
        }
        return res;
    }
};
```

## 2.位运算的最小翻转次数

给你三个正整数 a、b 和 c。

你可以对 a 和 b 的二进制表示进行位翻转操作，返回能够使按位或运算   a OR b == c  成立的最小翻转次数。

「位翻转操作」是指将一个数的二进制表示任何单个位上的 1 变成 0 或者 0 变成 1 。

输入：a = 2, b = 6, c = 5
输出：3
解释：翻转后 a = 1 , b = 4 , c = 5 使得 a OR b == c
示例 2：

输入：a = 4, b = 2, c = 7
输出：1
示例 3：

输入：a = 1, b = 2, c = 3
输出：0


提示：

1 <= a <= 10^9
1 <= b <= 10^9
1 <= c <= 10^9

### 解答

考点在于位运算，考虑每一位的变化就会很简单。分情况：

- a、b某一位上都是1，c为0，该位要变两次
- a、b某一位上都是1，c为1，不用变
- a、b某一位上为0与1，c为0，该位变1次
- a、b某一位上为0与1，c为1，不用变
- a、b某一位上都为0，c为1，该位变1次
- a、b某一位上都为0，c为0，不用变

清楚了这些规则后，发现除了不用变和唯一的一个变2次外，其他都是变1次，代码就好写了：

```C++
class Solution {
public:
    int minFlips(int a, int b, int c) {
        int i = 0, cnt = 0;
        int maxnum = max(a, b);
        while (1<<i <= maxnum || 1<<i <= c) {
            if (((a & 1<<i) | (b & 1<<i)) == (c & 1<<i)) {
                i++;
                continue;
            }
            if ((a & 1<<i) & (b & 1<<i)) cnt++;
            cnt++;
            i++;
        }
        return cnt;
    }
};
```

## 3.连通网络的操作次数

用以太网线缆将 n 台计算机连接成一个网络，计算机的编号从 0 到 n-1。线缆用 connections 表示，其中 connections[i] = [a, b] 连接了计算机 a 和 b。

网络中的任何一台计算机都可以通过网络直接或者间接访问同一个网络中其他任意一台计算机。

给你这个计算机网络的初始布线 connections，你可以拔开任意两台直连计算机之间的线缆，并用它连接一对未直连的计算机。请你计算并返回使所有计算机都连通所需的最少操作次数。如果不可能，则返回 -1 。 

 示例 1：

输入：n = 4, connections = [[0,1],[0,2],[1,2]]
输出：1
解释：拔下计算机 1 和 2 之间的线缆，并将它插到计算机 1 和 3 上。
示例 2：

输入：n = 6, connections = [[0,1],[0,2],[0,3],[1,2],[1,3]]
输出：2
示例 3：

输入：n = 6, connections = [[0,1],[0,2],[0,3],[1,2]]
输出：-1
解释：线缆数量不足。
示例 4：

输入：n = 5, connections = [[0,1],[0,2],[3,4],[2,3]]
输出：0


提示：

1 <= n <= 10^5
1 <= connections.length <= min(n*(n-1)/2, 10^5)
connections[i].length == 2
0 <= connections[i][0], connections[i][1] < n
connections[i][0] != connections[i][1]
没有重复的连接。
两台计算机不会通过多条线缆连接。

### 解答

这道题实际上是一个较为典型的求连通分量的题。

首先如果connections的长度小于n-1，那么一定不能连通网络。因为最理想的情况是所有的点全部排成一列，这样的话n个点之间至少需要有n-1个线才能全部连接起来。那么如何计算最少的操作次数呢？我们将这 n 台计算机看成 n 个节点，每一条线缆看成一条边，那么计算机和线缆就组成了一个无向图。假设这个无向图中有 k 个连通分量，连通分量就是指在一个集合中的任意节点可以互相连接，不在该集合中的节点不能达到该集合中的任意一个点。所以我们需要做的应该如下：

step 1：找到所有的连通分量

step 2：找到边的个数大于n-1的连通分量

step 3：将步骤2中的连通分量中的其中一条连接边连接到另一个连通分量中，构成一个新的连通分量

算法框架如下：

```
比较 n - 1 和 len(connections)：
如果前者大于后者，那么一定无解，返回 -1；
如果前者小于等于后者，那么我们统计出图中的连通分量数 k，返回 k - 1。
```

所以问题就变成了确定连通分量的大小，在这里介绍并查集和DFS方法。

首先是DFS方法，即首先将所有的节点设置为“未搜索”状态，然后遍历所有的节点，如果找到一个“未搜索”的节点，就从这个节点开始深度搜索，将其能达到的节点变成“已搜索”状态，这样就找到了一个连通分量。

```C++
class Solution {
private:
    vector<vector<int>> edges;
    vector<bool> used;

public:
    void dfs(int u) {
        used[u] = true;
        for (int v: edges[u]) {
            if (!used[v]) {
                dfs(v);
            }
        }
    }
    
    int makeConnected(int n, vector<vector<int>>& connections) {
        if (connections.size() < n - 1) {
            return -1;
        }

        edges.resize(n);
        for (auto&& c: connections) {
            edges[c[0]].push_back(c[1]);
            edges[c[1]].push_back(c[0]);
        }
        used.resize(n);

        int part = 0;
        for (int i = 0; i < n; ++i) {
            if (!used[i]) {
                ++part;
                dfs(i);
            }
        }      
        return part - 1;
    }
};
```

时间和空间复杂度都为$O(n+m)$，代码比较好理解。下面介绍一个并查集的模板方法，并查集可以参看我的[CS_Notes]()

```C++
class Solution {
public:
    static const int N = 1e5 + 7;
    int p[N];
    int find(int x) {
        if (p[x] != x) p[x] = find(p[x]);
        return p[x];
    }
    
    int makeConnected(int n, vector<vector<int>>& connections) {
        int Size = connections.size();

        for (int i = 0; i < n; i++) {
            p[i] = i;
        }

        int cnt = 0;
        for (int i = 0; i < Size; i ++) {
            int a = connections[i][0], b = connections[i][1];
            int pa = find(a), pb = find(b);
            if (pa == pb) cnt ++; // 多余的线
            else p[pa] = pb;
        }

        // 算出连通块的个数，用多余的线去连
        int connect = 0;
        for (int i = 0; i < n; i ++) {
            if (p[i] == i) connect ++;
        }

        if (connect - cnt > 1) return -1;
        
        return connect - 1;
    }
};
```

时间复杂度：$O(Nlog N + M)$，上述代码中的并查集只使用了路径压缩优化，没有使用按秩合并（将较小连通分量合并至较大的连通分量）优化。如果同时使用路径压缩优化和按秩合并优化，时间复杂度降低至 $O(N \alpha(N) + M)$。

空间复杂度：$O(N)$

## 4.二指输入的最小距离

![](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/11/leetcode_keyboard.png)

二指输入法定制键盘在 XY 平面上的布局如上图所示，其中每个大写英文字母都位于某个坐标处，例如字母 A 位于坐标 (0,0)，字母 B 位于坐标 (0,1)，字母 P 位于坐标 (2,3) 且字母 Z 位于坐标 (4,1)。

给你一个待输入字符串 word，请你计算并返回在仅使用两根手指的情况下，键入该字符串需要的最小移动总距离。坐标 (x1,y1) 和 (x2,y2) 之间的距离是 |x1 - x2| + |y1 - y2|。 

注意，两根手指的起始位置是零代价的，不计入移动总距离。你的两根手指的起始位置也不必从首字母或者前两个字母开始。

 示例 1：

输入：word = "CAKE"
输出：3
解释： 
使用两根手指输入 "CAKE" 的最佳方案之一是： 
手指 1 在字母 'C' 上 -> 移动距离 = 0 
手指 1 在字母 'A' 上 -> 移动距离 = 从字母 'C' 到字母 'A' 的距离 = 2 
手指 2 在字母 'K' 上 -> 移动距离 = 0 
手指 2 在字母 'E' 上 -> 移动距离 = 从字母 'K' 到字母 'E' 的距离  = 1 
总距离 = 3
示例 2：

输入：word = "HAPPY"
输出：6
解释： 
使用两根手指输入 "HAPPY" 的最佳方案之一是：
手指 1 在字母 'H' 上 -> 移动距离 = 0
手指 1 在字母 'A' 上 -> 移动距离 = 从字母 'H' 到字母 'A' 的距离 = 2
手指 2 在字母 'P' 上 -> 移动距离 = 0
手指 2 在字母 'P' 上 -> 移动距离 = 从字母 'P' 到字母 'P' 的距离 = 0
手指 1 在字母 'Y' 上 -> 移动距离 = 从字母 'A' 到字母 'Y' 的距离 = 4
总距离 = 6
示例 3：

输入：word = "NEW"
输出：3
示例 4：

输入：word = "YEAR"
输出：7


提示：

2 <= word.length <= 300
每个 word[i] 都是一个大写英文字母

### 解答

首先要考虑如何表示一个状态。下列因素是变动的：

- 需要走的下一个字符

- 当前左手指所在位置

- 当前右手指所在位置

所以一个状态是三维的：(i, left, right)。递推数组`dp[i][left][right]`表示到达第`i`步、左手指在`left`、右手指在`right`的最小步数，可以写出状态转移方程：

```
dp[i][ch][right] = min(dp[i][ch][right], dp[i][left][right] + dist(left, ch)); // 左指移动
dp[i][left][ch] = min(dp[i][left][ch], dp[i][left][right] + dist(right, ch)); // 右指移动
```

代码如下：

```C++
const int BASE = 26;

class Solution {
public:
    int minimumDistance(string word) {
        const int N = word.size();
        // dp[i][l][r]表示第i步、手指分别在left/right的最小步数
        vector<vector<vector<int>>> dp(N + 1, vector<vector<int>>(BASE, vector<int>(BASE, INT_MAX)));

        for (int i = 0; i < BASE; ++i) {
            for (int j = 0; j < BASE; ++j) {
                dp[0][i][j] = 0;
            }
        }

        for (int i = 1; i <= N; ++i) {
            int ch = word[i - 1] - 'A';
            for (int l = 0; l < BASE; ++l) {
                for (int r = 0; r < BASE; ++r) {
                    if (dp[i - 1][l][r] != INT_MAX) {
                        dp[i][ch][r] = min(dp[i][ch][r], dp[i - 1][l][r] + dist(l, ch)); // 左指走
                        dp[i][l][ch] = min(dp[i][l][ch], dp[i - 1][l][r] + dist(r, ch)); // 右指走
                    }
                }
            }
        }

        int res = INT_MAX;
        for (int l = 0; l < BASE; ++l) {
            for (int r = 0; r < BASE; ++r) {
                res = min(res, dp[N][l][r]);
            }
        }

        return res;
    }

    int dist(int from, int to) {
        int fx = from / 6, fy = from % 6;
        int tx = to / 6, ty = to % 6;
        return abs(fx - tx) + abs(fy - ty);
    }
};
```

