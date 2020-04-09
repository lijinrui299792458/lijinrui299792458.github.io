---
layout:     post
title:      Weekly Contest 183
subtitle:   Leetcode竞赛系列
date:       2020-04-09
author:     YunLambert
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - Algorithm

---

> Weekly Contest 182

竞赛链接：[https://leetcode-cn.com/contest/weekly-contest-183/](https://leetcode-cn.com/contest/weekly-contest-183)

## 非递增顺序的最小子序列

给你一个数组 `nums`，请你从中抽取一个子序列，满足该子序列的元素之和 **严格** 大于未包含在该子序列中的各元素之和。

如果存在多个解决方案，只需返回 **长度最小** 的子序列。如果仍然有多个解决方案，则返回 **元素之和最大** 的子序列。

与子数组不同的地方在于，「数组的子序列」不强调元素在原数组中的连续性，也就是说，它可以通过从数组中分离一些（也可能不分离）元素得到。

**注意**，题目数据保证满足所有约束条件的解决方案是 **唯一** 的。同时，返回的答案应当按 **非递增顺序** 排列。

 

**示例 1：**

```
输入：nums = [4,3,10,9,8]
输出：[10,9] 
解释：子序列 [10,9] 和 [10,8] 是最小的、满足元素之和大于其他各元素之和的子序列。但是 [10,9] 的元素之和最大。 
```

**示例 2：**

```
输入：nums = [4,4,7,6,7]
输出：[7,7,6] 
解释：子序列 [7,7] 的和为 14 ，不严格大于剩下的其他元素之和（14 = 4 + 4 + 6）。因此，[7,6,7] 是满足题意的最小子序列。注意，元素按非递增顺序返回。  
```

**示例 3：**

```
输入：nums = [6]
输出：[6]
```

 

**提示：**

- `1 <= nums.length <= 500`
- `1 <= nums[i] <= 100`

解答：

签到题

```C++
class Solution {
public:
    vector<int> minSubsequence(vector<int>& nums) {
        sort(nums.rbegin(), nums.rend());
        int sum = 0;
        for (int i = 0; i < nums.size(); ++i) {
            sum += nums[i];
        }
        vector<int> res;
        int t = 0;
        for (int i = 0; i < nums.size(); ++i) {
            t += nums[i];
            res.push_back(nums[i]);
            if (t > sum - t) break;
        }
        return res;
    }
};
```

## 将二进制表示减到1的步骤数

给你一个以二进制形式表示的数字 `s` 。请你返回按下述规则将其减少到 1 所需要的步骤数：

- 如果当前数字为偶数，则将其除以 2 。
- 如果当前数字为奇数，则将其加上 1 。

题目保证你总是可以按上述规则将测试用例变为 1 。

 

**示例 1：**

```
输入：s = "1101"
输出：6
解释："1101" 表示十进制数 13 。
Step 1) 13 是奇数，加 1 得到 14 
Step 2) 14 是偶数，除 2 得到 7
Step 3) 7  是奇数，加 1 得到 8
Step 4) 8  是偶数，除 2 得到 4  
Step 5) 4  是偶数，除 2 得到 2 
Step 6) 2  是偶数，除 2 得到 1  
```

**示例 2：**

```
输入：s = "10"
输出：1
解释："10" 表示十进制数 2 。
Step 1) 2 是偶数，除 2 得到 1 
```

**示例 3：**

```
输入：s = "1"
输出：0
```

 

**提示：**

- `1 <= s.length <= 500`
- `s` 由字符 `'0'` 或 `'1'` 组成。
- `s[0] == '1'`

解答：

以为是考察位运算，结果是考察字符串.....

```C++
class Solution {
public:
   int numSteps(string s) {
        int res = 0;
        while (1) {
            //cout<<s.size() << " "<<s[0]<< " "<<s<<" "<<res<<endl;
            if (s.size() == 1 && s[0] == '1') break;
            if (s[s.size() - 1] == '1') {
                int flag = 0;
                for (int i = s.size() - 1; i >= 0; --i) {
                    int x = s[i] - '0' + flag;
                    if (i == s.size() - 1) x += 1;
                    if (x > 1) {
                        flag = 1;
                        x = x%2;
                    } else {
                        flag = 0;
                    }
                    s[i] = '0' + x;
                }
                if (flag) s = "1" + s;
            }
            else s.pop_back();
            res++;
        }
        return res;
    }
};
```

有一个可以优化的地方，就是在确定s为奇数并+1后，步骤数直接加上s的末尾0的个数，因为在s加1变成偶数后，执行的操作为除2，只要末位不为0我们就能确保s仍然是偶数，所以对于一个奇数若最右边有x个连续的1 则需要x+1次操作对于偶数 只需要右移一位即可 需要1步操作 如此模拟即可。

优化后的代码如下：

```C++
int numSteps(string s) {
        int res = 0, n = s.size(),d = 0;
        for(int i = n-1; i >= 0; ){//从低位往高位开始遍历
            if(i==0&&s[i]=='1') break;//模拟完成
            if(s[i]=='1'){
                int x = 0;//最右边连续的1的个数
                while(i>=0&&s[i]=='1') i--,x++;//找到第一个非1位置
                res += x+1;//累加操作步数 
                if(i < 0) break;//即全为1的情况 进位在最前方 经过之前的+1 不断右移 答案就出来了，结束即可
                s[i] = '1';//之前的+1操作的效果
            }
            else res++, i--;//偶数时
        }
        return res;
    }
```



## 最长快乐字符串

如果字符串中不含有任何 `'aaa'`，`'bbb'` 或 `'ccc'` 这样的字符串作为子串，那么该字符串就是一个「快乐字符串」。

给你三个整数 `a`，`b` ，`c`，请你返回 **任意一个** 满足下列全部条件的字符串 `s`：

- `s` 是一个尽可能长的快乐字符串。
- `s` 中 **最多** 有`a` 个字母 `'a'`、`b` 个字母 `'b'`、`c` 个字母 `'c'` 。
- `s `中只含有 `'a'`、`'b'` 、`'c'` 三种字母。

如果不存在这样的字符串 `s` ，请返回一个空字符串 `""`。

 

**示例 1：**

```
输入：a = 1, b = 1, c = 7
输出："ccaccbcc"
解释："ccbccacc" 也是一种正确答案。
```

**示例 2：**

```
输入：a = 2, b = 2, c = 1
输出："aabbc"
```

**示例 3：**

```
输入：a = 7, b = 1, c = 0
输出："aabaa"
解释：这是该测试用例的唯一正确答案。
```

 

**提示：**

- `0 <= a, b, c <= 100`
- `a + b + c > 0`

解答：

比赛的时候的想法是每轮放置字符时优先先放剩余次数最多的, 如果上次放的2个字符和剩余个数最多的字符相同，则放置次多的字符。修修补补细节就成了这样：

```C++
#include <iostream>
#include <string>
#include <vector>
using namespace std;
typedef pair<char, int> PCI;
string longestDiverseString(int a, int b, int c) {
    vector<PCI> p;
    p.push_back({'a', a});
    p.push_back({'b', b});
    p.push_back({'c', c});
    sort(p.begin(), p.end(), [&](PCI p1, PCI p2) {
        if (p1.second == p2.second) return p1.first < p2.first;
        return p1.second > p2.second;
    });
    string res = "";
    int total = a + b + c;
    int index = 0, j = 0;
    bool flag = true;
    while (index < total && flag) {
        sort(p.begin(), p.end(), [&](PCI p1, PCI p2) {
            if (p1.second == p2.second) return p1.first < p2.first;
            return p1.second > p2.second;
        });

        if (res.back() == p[0].first) j = 1;
        else j = 0;

        int cnt = j == 0? 2 : 1;
        while (cnt--) {
            if (p[j].second > 0) {
                res += p[j].first;
                p[j].second--;
                index++;
            } else flag = false;
        }
    }
    //cout<<res<<endl;
    if (res.size() < 2) return res;
    int sub = 2;
    for (;sub < total; ++sub) {
        if (res[sub - 1] == res[sub] && res[sub] == res[sub-2]) break;
    }
    res = res.substr(0, sub);
    cout<<res<<endl;
    return res;
}

int main() {
    longestDiverseString(1, 1, 7);
    return 0;
}
```

很奇怪，在本地测试都可以，到线上提交的时候总是提示overflow，还没找到bug在哪里....

然后网上有大神的想法和我是类似的，不过代码简洁多了：

```C++
class Solution {
public:
    string longestDiverseString(int a, int b, int c) {
        vector<vector<char>> v;
        v.push_back({(char)a, 'a'});
        v.push_back({(char)b, 'b'});
        v.push_back({(char)c, 'c'});

        string res;
        while (res.size() < a + b + c) {            
            sort(v.rbegin(), v.rend());

            if ((res.size() > 0) && (res.back() == v[0][1])) {
                if (v[1][0]-- > 0) res.push_back(v[1][1]);
                else return res;
            } else {
                if (v[0][0]-- > 0) res.push_back(v[0][1]);
                if (v[0][0]-- > 0) res.push_back(v[0][1]);
            }
        }

        return res;
    }
};
```



也可以用贪心来做：

首先拿出 c 个 'a', 'b', 'c' 进行拼接。
再拿出 b-c 个 'a'，'b' 进行拼接。此时所有 'b'，'c' 都已拼接到答案中，仅剩 a-b 个 'a' 未拼接。
然后可以通过暴力枚举将尽可能多的 'a' 插入到答案中。

```C++
class Solution {
    bool check(int pos, const string &str, const string &ch) {
        string a = str;
        a.insert(pos, ch);
        for(int i = 0; i+2 < a.size(); i++) {
            if(a[i] == ch[0] && a[i+1] == ch[0] && a[i+2] == ch[0]) {
                return false;
            }
        }
        return true;
    }
public:
    string longestDiverseString(int a, int b, int c) {
        vector<pair<int, string>> vec;
        vec.push_back(make_pair(a, string("a")));
        vec.push_back(make_pair(b, string("b")));
        vec.push_back(make_pair(c, string("c")));
        sort(vec.begin(), vec.end());
        string str;
        while(vec[0].first > 0) {
            vec[0].first --;
            vec[1].first --;
            vec[2].first --;
            str += vec[2].second;
            str += vec[1].second;
            str += vec[0].second;
        }
        while(vec[1].first > 0) {
            vec[1].first --;
            vec[2].first --;
            str += vec[2].second;
            str += vec[1].second;
        }
        while(vec[2].first > 0) {
            bool flag = false;
            for(int i = 0; i <= str.size(); i++) {
                if(check(i, str, vec[2].second)) {
                    str.insert(i, vec[2].second);
                    flag = true;
                    break;
                }
            }
            if(flag == false) {
                break;
            }
            vec[2].first --;
        }
        return str;
    }
};
```



## 石子游戏3

Alice 和 Bob 用几堆石子在做游戏。几堆石子排成一行，每堆石子都对应一个得分，由数组 `stoneValue` 给出。

Alice 和 Bob 轮流取石子，**Alice** 总是先开始。在每个玩家的回合中，该玩家可以拿走剩下石子中的的前 **1、2 或 3 堆石子** 。比赛一直持续到所有石头都被拿走。

每个玩家的最终得分为他所拿到的每堆石子的对应得分之和。每个玩家的初始分数都是 **0** 。比赛的目标是决出最高分，得分最高的选手将会赢得比赛，比赛也可能会出现平局。

假设 Alice 和 Bob 都采取 **最优策略** 。如果 Alice 赢了就返回 *"Alice"* *，*Bob 赢了就返回 *"Bob"，*平局（分数相同）返回 *"Tie"* 。

 

**示例 1：**

```
输入：values = [1,2,3,7]
输出："Bob"
解释：Alice 总是会输，她的最佳选择是拿走前三堆，得分变成 6 。但是 Bob 的得分为 7，Bob 获胜。
```

**示例 2：**

```
输入：values = [1,2,3,-9]
输出："Alice"
解释：Alice 要想获胜就必须在第一个回合拿走前三堆石子，给 Bob 留下负分。
如果 Alice 只拿走第一堆，那么她的得分为 1，接下来 Bob 拿走第二、三堆，得分为 5 。之后 Alice 只能拿到分数 -9 的石子堆，输掉比赛。
如果 Alice 拿走前两堆，那么她的得分为 3，接下来 Bob 拿走第三堆，得分为 3 。之后 Alice 只能拿到分数 -9 的石子堆，同样会输掉比赛。
注意，他们都应该采取 最优策略 ，所以在这里 Alice 将选择能够使她获胜的方案。
```

**示例 3：**

```
输入：values = [1,2,3,6]
输出："Tie"
解释：Alice 无法赢得比赛。如果她决定选择前三堆，她可以以平局结束比赛，否则她就会输。
```

**示例 4：**

```
输入：values = [1,2,3,-1,-2,-3,7]
输出："Alice"
```

**示例 5：**

```
输入：values = [-1,-2,-3]
输出："Tie"
```

 

**提示：**

- `1 <= values.length <= 50000`
- `-1000 <= values[i] <= 1000`

解答：

这篇解看起来很容易理解：[https://leetcode-cn.com/problems/stone-game-iii/solution/ling-he-dui-shou-cai-qu-zui-you-de-fen-zui-shao-sh/](https://leetcode-cn.com/problems/stone-game-iii/solution/ling-he-dui-shou-cai-qu-zui-you-de-fen-zui-shao-sh/)

