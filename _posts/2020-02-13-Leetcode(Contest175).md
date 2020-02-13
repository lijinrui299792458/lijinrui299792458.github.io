---
layout:     post
title:      Weekly Contest 175
subtitle:   Leetcode竞赛系列
date:       2020-02-13
author:     YunLambert
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - Algorithm

---

> Weekly Contest 175

竞赛链接：[https://leetcode-cn.com/contest/weekly-contest-175/](https://leetcode-cn.com/contest/weekly-contest-175/)

## 检查整数及其两倍数是否存在

给你一个整数数组 arr，请你检查是否存在两个整数 N 和 M，满足 N 是 M 的两倍（即，N = 2 * M）。

更正式地，检查是否存在两个下标 i 和 j 满足：

i != j
0 <= i, j < arr.length
arr[i] == 2 * arr[j]


示例 1：

输入：arr = [10,2,5,3]
输出：true
解释：N = 10 是 M = 5 的两倍，即 10 = 2 * 5 。
示例 2：

输入：arr = [7,1,14,11]
输出：true
解释：N = 14 是 M = 7 的两倍，即 14 = 2 * 7 。

解答：

直接暴力做就行了：

```C++
class Solution {
public:
    bool checkIfExist(vector<int>& arr) {
        for (int i = 0; i < arr.size(); ++i) {
            for (int j = 0; j < arr.size(); ++j) {
                if (i == j) continue;
                if (arr[i] == 2*arr[j]) return true;
            }
        }
        return false;
    }
};
```

## 制造字母异位词的最小步骤数

给你两个长度相等的字符串 s 和 t。每一个步骤中，你可以选择将 t 中的 任一字符 替换为 另一个字符。

返回使 t 成为 s 的字母异位词的最小步骤数。

字母异位词 指字母相同，但排列不同的字符串。

 

示例 1：

输出：s = "bab", t = "aba"
输出：1
提示：用 'b' 替换 t 中的第一个 'a'，t = "bba" 是 s 的一个字母异位词。
示例 2：

输出：s = "leetcode", t = "practice"
输出：5
提示：用合适的字符替换 t 中的 'p', 'r', 'a', 'i' 和 'c'，使 t 变成 s 的字母异位词。

解答：

用一个map存储数据就可以了。

```C++
class Solution {
public:
    int minSteps(string s, string t) {
        map<char, int> m;
        for (int i = 0; i < s.size(); ++i) m[s[i]]++;
        for (int i = 0; i < t.size(); ++i) if (m[t[i]]) m[t[i]]--;
        int res = 0;
        for (auto it = m.begin(); it!=m.end(); it++) {
            if (it->second!=0) res += it->second;
        }
        return res;
    }
};
```

## 推文计数

请你实现一个能够支持以下两种方法的推文计数类 TweetCounts：

1. recordTweet(string tweetName, int time)

记录推文发布情况：用户 tweetName 在 time（以 秒 为单位）时刻发布了一条推文。
2. getTweetCountsPerFrequency(string freq, string tweetName, int startTime, int endTime)

返回从开始时间 startTime（以 秒 为单位）到结束时间 endTime（以 秒 为单位）内，每 分 minute，时 hour 或者 日 day （取决于 freq）内指定用户 tweetName 发布的推文总数。
freq 的值始终为 分 minute，时 hour 或者 日 day 之一，表示获取指定用户 tweetName 发布推文次数的时间间隔。
第一个时间间隔始终从 startTime 开始，因此时间间隔为 [startTime, startTime + delta*1>,  [startTime + delta*1, startTime + delta*2>, [startTime + delta*2, startTime + delta*3>, ... , [startTime + delta*i, min(startTime + delta*(i+1), endTime + 1)>，其中 i 和 delta（取决于 freq）都是非负整数。


示例：

输入：
["TweetCounts","recordTweet","recordTweet","recordTweet","getTweetCountsPerFrequency","getTweetCountsPerFrequency","recordTweet","getTweetCountsPerFrequency"]
[[],["tweet3",0],["tweet3",60],["tweet3",10],["minute","tweet3",0,59],["minute","tweet3",0,60],["tweet3",120],["hour","tweet3",0,210]]

输出：
[null,null,null,null,[2],[2,1],null,[4]]

解答：

这道题当时想了很久，比较重要的就是时间区间是左闭右开，且是$min(startTime + delta*(i+1), endTime + 1)$

```C++
class TweetCounts {
public:
    TweetCounts() {
        
    }    

    void recordTweet(string tweetName, int time) 
    {
			record[tweetName][time]++;
    }
    
    vector<int> getTweetCountsPerFrequency(string freq, string tweetName, int startTime, int endTime)
    {
		int f = 1;
    if (freq == "minute") f = 60;
    else if (freq == "hour") f = 3600;
    else f = 3600*24;

		vector<int> ans;
		int t = startTime;
		while (t <= endTime)
		{
			auto bg = record[tweetName].lower_bound(t);
			auto ed = record[tweetName].upper_bound(min(t + f - 1, endTime));
			int cnt = 0;
			for (auto it = bg; it != ed; it++)
			{
				cnt += it->second;
			}
			ans.push_back(cnt);
			t += f;
		}
		return ans;
    }

private:
	unordered_map<string, map<int, int>> record;
};


/**
 * Your TweetCounts object will be instantiated and called as such:
 * TweetCounts* obj = new TweetCounts();
 * obj->recordTweet(tweetName,time);
 * vector<int> param_2 = obj->getTweetCountsPerFrequency(freq,tweetName,startTime,endTime);
 */
```

## 参加考试的最大学生数

给你一个 `m * n` 的矩阵 `seats` 表示教室中的座位分布。如果座位是坏的（不可用），就用 `'#'` 表示；否则，用 `'.'` 表示。

学生可以看到左侧、右侧、左上、右上这四个方向上紧邻他的学生的答卷，但是看不到直接坐在他前面或者后面的学生的答卷。请你计算并返回该考场可以容纳的一起参加考试且无法作弊的最大学生人数。

学生必须坐在状况良好的座位上。

**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/02/09/image.png)

```
输入：seats = [["#",".","#","#",".","#"],
              [".","#","#","#","#","."],
              ["#",".","#","#",".","#"]]
输出：4
解释：教师可以让 4 个学生坐在可用的座位上，这样他们就无法在考试中作弊。 
```

**示例 2：**

```
输入：seats = [[".","#"],
              ["#","#"],
              ["#","."],
              ["#","#"],
              [".","#"]]
输出：3
解释：让所有学生坐在可用的座位上。
```

**示例 3：**

```
输入：seats = [["#",".",".",".","#"],
              [".","#",".","#","."],
              [".",".","#",".","."],
              [".","#",".","#","."],
              ["#",".",".",".","#"]]
输出：10
解释：让学生坐在第 1、3 和 5 列的可用座位上。
```

 **提示：**

- `seats` 只包含字符 `'.' 和``'#'`
- `m == seats.length`
- `n == seats[i].length`
- `1 <= m <= 8`
- `1 <= n <= 8`

解答：

这道题没有做出来，直接等答案了。自己最初的想法是类似八皇后问题那样的，一层一层的递归进行判断，但是后来思路觉得有些复杂就没有继续做下去。

这道题可以用状压DP来做，以$dp[i][bits]$来表示前i行中，第i行作为情况为bits的最大答案，因为数据范围为8位，所以就用bit表示最为方便。

在这里直接放出代码 ，解析可以直接看[这里](https://leetcode-cn.com/problems/maximum-students-taking-exam/solution/xiang-jie-ya-suo-zhuang-tai-dong-tai-gui-hua-jie-f/)

```C++
class Solution {
public:
    int maxStudents(vector<vector<char>>& seats) {
        int m=seats.size();
        int n=seats[0].size();
        vector<vector<int>> dp(m+1,vector<int>(1<<n));  //初始化

    for(int row=1;row<=m;row++){
        for(int s=0;s<(1<<n);s++){//遍历 2^n 个状态
            bitset<8> bs(s);//记录对应状态的bit位
            bool ok=true;
            for(int j=0;j<n;j++){
                if((bs[j] && seats[row-1][j]=='#') || (j<n-1 && bs[j] && bs[j+1])){//不能坐在坏椅子上也不能在同一行相邻坐
                    ok=false;
                    break;
                }
            }
            if(!ok){
                dp[row][s]=-1;//说明坐在坏椅子上或相邻坐了，该状态舍弃
                continue;
            }
            for(int last=0;last<(1<<n);last++){//找到一种当前行的可行状态后，遍历上一行的所有状态
                if(dp[row-1][last]==-1)//上一行的状态被舍弃了，那就直接下一个状态
                    continue;
                bitset<8> lbs(last);
                bool flag=true;
                for(int j=0;j<n;j++){
                    if(lbs[j] && ((j>0 && bs[j-1]) || (j<n-1 && bs[j+1]))){//如果找到的这个上一行状态的j位置坐了人，
                        flag=false;                                    //下一行的j+1位置或j-1位置也坐了人，那么该状态不合法，舍弃
                        break;
                    }
                }
                if(flag){//flag为真说明这个last状态的每个位置都合法
                    dp[row][s]=max(dp[row][s],dp[row-1][last]+(int)bs.count());//转移方程
                }
            }

        }
    }
    int res=0;
    for(int i=0;i<(1<<n);i++){//在最后一行的所有状态中找出最大的
        if(dp[m][i]>res){
            res=dp[m][i];
        }
    }
    return res;
    }
};
```

## 总结

前三题都不算太难，第三题理解题意的时候出现了一点问题，出了一点小bug。但是第四题是真的不会，功力还是不够，动规上得下功夫了。



