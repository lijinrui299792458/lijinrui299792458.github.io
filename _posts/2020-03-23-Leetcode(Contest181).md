---
layout:     post
title:      Weekly Contest 181
subtitle:   Leetcode竞赛系列
date:       2020-03-23
author:     YunLambert
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - Algorithm

---

> Weekly Contest 181

竞赛链接：[https://leetcode-cn.com/contest/weekly-contest-181/](https://leetcode-cn.com/contest/weekly-contest-181)

## 按既定顺序创建目标数组

给你两个整数数组 `nums` 和 `index`。你需要按照以下规则创建目标数组：

- 目标数组 `target` 最初为空。
- 按从左到右的顺序依次读取 `nums[i]` 和 `index[i]`，在 `target` 数组中的下标 `index[i]` 处插入值 `nums[i]` 。
- 重复上一步，直到在 `nums` 和 `index` 中都没有要读取的元素。

请你返回目标数组。

题目保证数字插入位置总是存在。

 

**示例 1：**

```
输入：nums = [0,1,2,3,4], index = [0,1,2,2,1]
输出：[0,4,1,3,2]
解释：
nums       index     target
0            0        [0]
1            1        [0,1]
2            2        [0,1,2]
3            2        [0,1,3,2]
4            1        [0,4,1,3,2]
```

**示例 2：**

```
输入：nums = [1,2,3,4,0], index = [0,1,2,3,0]
输出：[0,1,2,3,4]
解释：
nums       index     target
1            0        [1]
2            1        [1,2]
3            2        [1,2,3]
4            3        [1,2,3,4]
0            0        [0,1,2,3,4]
```

**示例 3：**

```
输入：nums = [1], index = [0]
输出：[1]
```

 

**提示：**

- `1 <= nums.length, index.length <= 100`
- `nums.length == index.length`
- `0 <= nums[i] <= 100`
- `0 <= index[i] <= i`

解答：

考察stl的用法，签到题

```C++
class Solution {
public:
    vector<int> createTargetArray(vector<int>& nums, vector<int>& index) {
        vector<int> t(nums.size(), -1);
        for (int i = 0; i < nums.size(); ++i) {
            if (t[index[i]] == -1) t[index[i]] = nums[i];
            else {
                auto it = t.begin();
                t.insert(it+index[i], nums[i]);
            }
        }
        vector<int> target(nums.size());
        std::copy(t.begin(), t.begin()+nums.size(), target.begin());
        return target;
    }
};
```

## 四因数

给你一个整数数组 `nums`，请你返回该数组中恰有四个因数的这些整数的各因数之和。

如果数组中不存在满足题意的整数，则返回 `0` 。

 

**示例：**

```
输入：nums = [21,4,7]
输出：32
解释：
21 有 4 个因数：1, 3, 7, 21
4 有 3 个因数：1, 2, 4
7 有 2 个因数：1, 7
答案仅为 21 的所有因数的和。
```

 

**提示：**

- `1 <= nums.length <= 10^4`
- `1 <= nums[i] <= 10^5`

解答：

写一个找4个因数的子函数，其中要注意的是eg 1 2 2 4这种例子里面会出现22个2但是计算的时候只会出现一次，所以要特判一下是否会相等的情况：

```C++
class Solution {
public:
    int get4DivisorsSum(int x) {
        int cnt = 0, sum = 0;
        for (int i = 1; i*i <= x; ++i) {
            if (x % i == 0) {
                sum += i;
                if (i != x/i) {
                    sum += x/i;
                    cnt += 2;
                } else cnt++;
                //cout<<i<<" "<<x/i<<endl;
            }
        }
        if (cnt == 4) return sum;
        return -1;
    }
    int sumFourDivisors(vector<int>& nums) {
        int res = 0;
        for (int i = 0; i < nums.size(); ++i) {
            int t = get4DivisorsSum(nums[i]);
            cout<<t<<endl;
            if (t == -1) continue;
            res += t;
        }
        return res;
    }
};
```

## 检查网格中是否存在有效路径

给你一个 *m* x *n* 的网格 `grid`。网格里的每个单元都代表一条街道。`grid[i][j]` 的街道可以是：

- **1** 表示连接左单元格和右单元格的街道。
- **2** 表示连接上单元格和下单元格的街道。
- **3** 表示连接左单元格和下单元格的街道。
- **4** 表示连接右单元格和下单元格的街道。
- **5** 表示连接左单元格和上单元格的街道。
- **6** 表示连接右单元格和上单元格的街道。

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/03/21/main.png)

你最开始从左上角的单元格 `(0,0)` 开始出发，网格中的「有效路径」是指从左上方的单元格 `(0,0)` 开始、一直到右下方的 `(m-1,n-1)` 结束的路径。**该路径必须只沿着街道走**。

**注意：**你 **不能** 变更街道。

如果网格中存在有效的路径，则返回 `true`，否则返回 `false` 。

 

**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/03/21/e1.png)

```
输入：grid = [[2,4,3],[6,5,2]]
输出：true
解释：如图所示，你可以从 (0, 0) 开始，访问网格中的所有单元格并到达 (m - 1, n - 1) 。
```

**示例 2：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/03/21/e2.png)

```
输入：grid = [[1,2,1],[1,2,1]]
输出：false
解释：如图所示，单元格 (0, 0) 上的街道没有与任何其他单元格上的街道相连，你只会停在 (0, 0) 处。
```

**示例 3：**

```
输入：grid = [[1,1,2]]
输出：false
解释：你会停在 (0, 1)，而且无法到达 (0, 2) 。
```

**示例 4：**

```
输入：grid = [[1,1,1,1,1,1,3]]
输出：true
```

**示例 5：**

```
输入：grid = [[2],[2],[2],[2],[2],[2],[6]]
输出：true
```

 

**提示：**

- `m == grid.length`
- `n == grid[i].length`
- `1 <= m, n <= 300`
- `1 <= grid[i][j] <= 6`

解答：

主要难点在于定义一个pipe数组，然后直接用dfs搞就行了。`pipe[t][dir]`的定义是这样的，t表示目前所在拼图的种类（1-6），即grid中对应的数字。dir则是表示上一个拼图到此时这个拼图的可能的方向(0，1，2，3)，分别表示下、右、上、左 4个方向(*可以理解为目前所在拼图的入口*)。如拼图1就可以表示为dir = 1 或 3. 而`pipe[t][dir]`的值则是表示此时这个拼图到下一个拼图的可能方向(*可以理解为目前所在拼图的出口*)。需要注意的是此时的拼图的出口，就是下一个拼图的入口，这点需要注意一下，所以dfs的时候`pipe[t][dir]`就会变成下一次dfs时候的入口值！

```C++
class Solution {
public:
    int dx[4] = {1, 0, -1, 0};
    int dy[4] = {0, 1, 0, -1};
    int pipe[7][4]={{-1,-1,-1,-1},{-1,1,-1,3},{0,-1,2,-1},{-1,0,3,-1},{-1,-1,1,0},{3,2,-1,-1},{1,-1,-1,2}};
    bool dfs(int x, int y, int dir, vector<vector<int>>& grid) {
        if (x == grid.size()-1 && y == grid[0].size()-1) return true;
        int nx = x + dx[dir], ny = y + dy[dir];
        if (nx < 0 || nx > grid.size() - 1 || ny < 0 || ny > grid[0].size()-1) return false;
        int t = grid[nx][ny];
        if (pipe[t][dir] != -1) return dfs(nx, ny, pipe[t][dir], grid);
        else return false;
    }

    bool hasValidPath(vector<vector<int>>& grid) {
        int t = grid[0][0];
        for (int i = 0; i < 4; ++i) {
            if (pipe[t][i] != -1 && dfs(0, 0, pipe[t][i], grid)) return true;
        }
        return false;
    }
};
```





## 最长快乐前缀

「快乐前缀」是在原字符串中既是 **非空** 前缀也是后缀（不包括原字符串自身）的字符串。

给你一个字符串 `s`，请你返回它的 **最长快乐前缀**。

如果不存在满足题意的前缀，则返回一个空字符串。

 

**示例 1：**

```
输入：s = "level"
输出："l"
解释：不包括 s 自己，一共有 4 个前缀（"l", "le", "lev", "leve"）和 4 个后缀（"l", "el", "vel", "evel"）。最长的既是前缀也是后缀的字符串是 "l" 。
```

**示例 2：**

```
输入：s = "ababab"
输出："abab"
解释："abab" 是最长的既是前缀也是后缀的字符串。题目允许前后缀在原字符串中重叠。
```

**示例 3：**

```
输入：s = "leetcodeleet"
输出："leet"
```

**示例 4：**

```
输入：s = "a"
输出：""
```

 

**提示：**

- `1 <= s.length <= 10^5`
- `s` 只含有小写英文字母

解答：

解法1：

实际上本题就是求解KMP的next数组

```C++
class Solution {
public:
    string longestPrefix(string s) {
        int n = s.size();
        vector<int> ne(n, -1);
        for (int i = 1; i < n; ++i) {
            int j = ne[i - 1];
            while (j != -1 && s[j+1] != s[i]) {
                j = ne[j];
            }
            if (s[j+1] == s[i]) ne[i] = j + 1;
        }
        int index = ne[n-1];
        return s.substr(0, index+1);
    }
};
```

解法2：

由于题目所给的数据范围发生哈希冲突的可能性较低，所以可以用26进制计算前缀和 和 后缀和，如果两个和相等则说明对应的子字符串相同：

```C++
class Solution {
    long long Q[100100],H[100100],P[100100];
    const long long M=1000000007;
    int n;

void power(int n){//预处理一个26的幂次表
    P[0]=1;
    for(int i=1;i<=n;i++)
    P[i]=(P[i-1]*26)%M;
}
public:
    string longestPrefix(string s) {
        n=s.length();
        power(n);

        Q[0]=s[0]-'a';//前缀数组
        for(int i=1;i<n;i++)
            Q[i]=(Q[i-1]*26+s[i]-'a')%M;

        H[0]=s[n-1]-'a';//后缀数组
        for(int i=n-2;i>=0;i--)
            H[n-1-i] = (H[n-i-2]+(s[i]-'a')*P[n-1-i])%M;

        int ans=-1;
        for(int i=n-2;i>=0;i--)
            if(Q[i]==H[i]){
                ans=i;
                break;
        }

        if(ans<0) return "";
        return s.substr(0,ans+1);
    }
};
```

## 总结

最近打的比赛比较多，一时间挤在一起都没有时间总结。以后一打完就要立刻总结呀！

