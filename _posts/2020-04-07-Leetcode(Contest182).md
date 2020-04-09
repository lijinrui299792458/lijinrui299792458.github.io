---
layout:     post
title:      Weekly Contest 182
subtitle:   Leetcode竞赛系列
date:       2020-04-07
author:     YunLambert
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - Algorithm

---

> Weekly Contest 182

竞赛链接：[https://leetcode-cn.com/contest/weekly-contest-182/](https://leetcode-cn.com/contest/weekly-contest-182)

## 找出数组中的幸运数

在整数数组中，如果一个整数的出现频次和它的数值大小相等，我们就称这个整数为「幸运数」。

给你一个整数数组 `arr`，请你从中找出并返回一个幸运数。

- 如果数组中存在多个幸运数，只需返回 **最大** 的那个。
- 如果数组中不含幸运数，则返回 **-1** 。

 

**示例 1：**

```
输入：arr = [2,2,3,4]
输出：2
解释：数组中唯一的幸运数是 2 ，因为数值 2 的出现频次也是 2 。
```

**示例 2：**

```
输入：arr = [1,2,2,3,3,3]
输出：3
解释：1、2 以及 3 都是幸运数，只需要返回其中最大的 3 。
```

**示例 3：**

```
输入：arr = [2,2,2,3,3]
输出：-1
解释：数组中不存在幸运数。
```

**示例 4：**

```
输入：arr = [5]
输出：-1
```

**示例 5：**

```
输入：arr = [7,7,7,7,7,7,7]
输出：7
```

 

**提示：**

- `1 <= arr.length <= 500`
- `1 <= arr[i] <= 500`

解答：

签到题：

```C++
class Solution {
public:
    int findLucky(vector<int>& arr) {
        map<int, int> m;
        int res = -1;
        for (int i = 0; i < arr.size(); ++i) m[arr[i]]++;
        for (auto it : m) {
            if (it.first == it.second) res = max(res, it.first);
        }
        return res;
    }
};
```

## 统计作战单位数

 `n` 名士兵站成一排。每个士兵都有一个 **独一无二** 的评分 `rating` 。

每 **3** 个士兵可以组成一个作战单位，分组规则如下：

- 从队伍中选出下标分别为 `i`、`j`、`k` 的 3 名士兵，他们的评分分别为 `rating[i]`、`rating[j]`、`rating[k]`
- 作战单位需满足： `rating[i] < rating[j] < rating[k]` 或者 `rating[i] > rating[j] > rating[k]` ，其中  `0 <= i < j < k < n`

请你返回按上述条件可以组建的作战单位数量。每个士兵都可以是多个作战单位的一部分。

 

**示例 1：**

```
输入：rating = [2,5,3,4,1]
输出：3
解释：我们可以组建三个作战单位 (2,3,4)、(5,4,1)、(5,3,1) 。
```

**示例 2：**

```
输入：rating = [2,1,3]
输出：0
解释：根据题目条件，我们无法组建作战单位。
```

**示例 3：**

```
输入：rating = [1,2,3,4]
输出：4
```

 

**提示：**

- `n == rating.length`
- `1 <= n <= 200`
- `1 <= rating[i] <= 10^5`

解答：

直接用暴搜竟然也能过....

```C++
class Solution {
public:
    int numTeams(vector<int>& rating) {
        int res = 0;
        for (int i = 0; i < rating.size()-2; ++i) {
            for (int j = i + 1; j < rating.size() - 1; ++j) {
                for (int k = j + 1; k < rating.size(); ++k) {
                    if ((rating[i] < rating[j] && rating [j] < rating[k]) || (rating[i] > rating[j] && rating[j] > rating[k]))
                        res ++;
                }
            }
        }
        return res;
    }
};
```

但是复杂度太高了，其他的思路如：

![](https://raw.githubusercontent.com/YunLambert/Blog_Pictures/master/img/20200407145146.png)

这样可以将复杂度降为$O(n^{2})$：

```C+
class Solution {
public:
    int numTeams(vector<int>& rating) {
        int n = rating.size();
        int ans = 0;
        // 枚举三元组中的 j
        for (int j = 1; j < n - 1; ++j) {
            int iless = 0, imore = 0;
            int kless = 0, kmore = 0;
            for (int i = 0; i < j; ++i) {
                if (rating[i] < rating[j]) {
                    ++iless;
                }
                // 注意这里不能直接写成 else
                // 因为可能有评分相同的情况
                else if (rating[i] > rating[j]) {
                    ++imore;
                }
            }
            for (int k = j + 1; k < n; ++k) {
                if (rating[k] < rating[j]) {
                    ++kless;
                }
                else if (rating[k] > rating[j]) {
                    ++kmore;
                }
            }
            ans += iless * kmore + imore * kless;
        }
        return ans;
    }
};
```

还有一种压缩的方法：

![](https://raw.githubusercontent.com/YunLambert/Blog_Pictures/master/img/20200407145441.png)

```C++
class Solution {
public:
    static constexpr int MAX_N = 200 + 5;

    int c[MAX_N];
    vector <int> disc;
    vector <int> iLess, iMore, kLess, kMore;

    int lowbit(int x) {
        return x & (-x);
    }

    void add(int p, int v) {
        while (p < MAX_N) {
            c[p] += v;
            p += lowbit(p);
        }
    }

    int get(int p) {
        int r = 0;
        while (p > 0) {
            r += c[p];
            p -= lowbit(p);
        }
        return r;
    }

    int numTeams(vector<int>& rating) {
        disc = rating;
        disc.push_back(-1);
        sort(disc.begin(), disc.end());
        auto getId = [&] (int target) {
            return lower_bound(disc.begin(), disc.end(), target) - disc.begin();
        };


        iLess.resize(rating.size());
        iMore.resize(rating.size());
        kLess.resize(rating.size());
        kMore.resize(rating.size());

        for (int i = 0; i < rating.size(); ++i) {
            auto id = getId(rating[i]);
            iLess[i] = get(id);
            iMore[i] = get(201) - get(id); 
            add(id, 1);
        }

        memset(c, 0, sizeof c);
        for (int i = rating.size() - 1; i >= 0; --i) {
            auto id = getId(rating[i]);
            kLess[i] = get(id);
            kMore[i] = get(201) - get(id); 
            add(id, 1);
        }
        
        int ans = 0;
        for (unsigned i = 0; i < rating.size(); ++i) {
            ans += iLess[i] * kMore[i] + iMore[i] * kLess[i];
        }

        return ans;
    }
};
```

## 设计地铁系统

请你实现一个类 `UndergroundSystem` ，它支持以下 3 种方法：

1.` checkIn(int id, string stationName, int t)`

- 编号为 `id` 的乘客在 `t` 时刻进入地铁站 `stationName` 。
- 一个乘客在同一时间只能在一个地铁站进入或者离开。

2.` checkOut(int id, string stationName, int t)`

- 编号为 `id` 的乘客在 `t` 时刻离开地铁站 `stationName` 。

\3. `getAverageTime(string startStation, string endStation)` 

- 返回从地铁站 `startStation` 到地铁站 `endStation` 的平均花费时间。
- 平均时间计算的行程包括当前为止所有从 `startStation` **直接到达** `endStation` 的行程。
- 调用 `getAverageTime` 时，询问的路线至少包含一趟行程。

你可以假设所有对 `checkIn` 和 `checkOut` 的调用都是符合逻辑的。也就是说，如果一个顾客在 **t1** 时刻到达某个地铁站，那么他离开的时间 **t2** 一定满足 **t2 > t1** 。所有的事件都按时间顺序给出。

 

**示例：**

```
输入：
["UndergroundSystem","checkIn","checkIn","checkIn","checkOut","checkOut","checkOut","getAverageTime","getAverageTime","checkIn","getAverageTime","checkOut","getAverageTime"]
[[],[45,"Leyton",3],[32,"Paradise",8],[27,"Leyton",10],[45,"Waterloo",15],[27,"Waterloo",20],[32,"Cambridge",22],["Paradise","Cambridge"],["Leyton","Waterloo"],[10,"Leyton",24],["Leyton","Waterloo"],[10,"Waterloo",38],["Leyton","Waterloo"]]

输出：
[null,null,null,null,null,null,null,14.0,11.0,null,11.0,null,12.0]

解释：
UndergroundSystem undergroundSystem = new UndergroundSystem();
undergroundSystem.checkIn(45, "Leyton", 3);
undergroundSystem.checkIn(32, "Paradise", 8);
undergroundSystem.checkIn(27, "Leyton", 10);
undergroundSystem.checkOut(45, "Waterloo", 15);
undergroundSystem.checkOut(27, "Waterloo", 20);
undergroundSystem.checkOut(32, "Cambridge", 22);
undergroundSystem.getAverageTime("Paradise", "Cambridge");       // 返回 14.0。从 "Paradise"（时刻 8）到 "Cambridge"(时刻 22)的行程只有一趟
undergroundSystem.getAverageTime("Leyton", "Waterloo");          // 返回 11.0。总共有 2 躺从 "Leyton" 到 "Waterloo" 的行程，编号为 id=45 的乘客出发于 time=3 到达于 time=15，编号为 id=27 的乘客于 time=10 出发于 time=20 到达。所以平均时间为 ( (15-3) + (20-10) ) / 2 = 11.0
undergroundSystem.checkIn(10, "Leyton", 24);
undergroundSystem.getAverageTime("Leyton", "Waterloo");          // 返回 11.0
undergroundSystem.checkOut(10, "Waterloo", 38);
undergroundSystem.getAverageTime("Leyton", "Waterloo");          // 返回 12.0
```

 

**提示：**

- 总共最多有 `20000` 次操作。
- `1 <= id, t <= 10^6`
- 所有的字符串包含大写字母，小写字母和数字。
- `1 <= stationName.length <= 10`
- 与标准答案误差在 `10^-5` 以内的结果都视为正确结果。

解答：

自己的思路是用一个Node来集成所有的信息，如果乘客有新的行程直接更新就可以，但是发现test没有过，原因是：假设有一个乘客多次checkin checkout，只能记录这个乘客最后一次的时间和目的地，这样就丢失了之前的信息，所以会报错。所以临时加了一个complete表示这个对象是否已经完成了，如果同时具有起始地和目标地，就可以新建新的对象，核对答案后没有问题；但是不幸的是提交的时候超时了，答案是用嵌套map来做的，看来用vector存储结构体的效率不如红黑树的效率。先放上比赛的时候的代码(未AC)：

```C++
class UndergroundSystem {
public:
    struct Node {
        bool complete = false;
        int id;
        string startName = "";
        string endName ="";
        int t1 = 0;
        int t2 = 0;
    };

    vector<Node> v;
    UndergroundSystem() {

    }
    
    void checkIn(int id, string stationName, int t) {
        bool flag = false;
        for (int i = 0; i < v.size(); ++i) {
            auto it = &v[i];
            if (it->id == id && !it->complete) {
                it->startName = stationName;
                it->t1 = t;
                flag = true;
                break;
            }
        }
        if (!flag) {
            Node x;
            x.id = id, x.startName = stationName, x.t1 = t;
            v.push_back(x);
        }
       
    }
    
    void checkOut(int id, string stationName, int t) {
        for (int i = 0; i < v.size(); ++i) {
            auto it = &v[i];
            if (it->id == id && !it->complete) {
                it->endName = stationName;
                it->t2 = t;
                it->complete = true;
                break;
            }
        }
    }
    
    double getAverageTime(string startStation, string endStation) {
        long long sum = 0;
        double cnt = 0.0;
        for (auto it : v) {
            if (it.startName == startStation && it.endName == endStation && it.t2 > it.t1) {
                sum += it.t2 - it.t1;
                cnt++;
            }
        }
       return sum/cnt;
    }
};

/**
 * Your UndergroundSystem object will be instantiated and called as such:
 * UndergroundSystem* obj = new UndergroundSystem();
 * obj->checkIn(id,stationName,t);
 * obj->checkOut(id,stationName,t);
 * double param_3 = obj->getAverageTime(startStation,endStation);
 */
```

然后给出正解：

```C++
class UndergroundSystem {
public:
	//第一个map的key为起始站(string)，
	//第二个map的key为终点站，val的第一个int为所有 以此为起终点 的时间累计，第二个int为 以此为起终点 的次数
	unordered_map<string, unordered_map<string, pair<int, int> > > mp;

	//key为乘客id，val的string为起始站，int为到达起始站的时刻
	unordered_map<int, pair<string, int> > mmp;
	UndergroundSystem() {}

	void checkIn(int id, string stationName, int t) {
		mmp[id].first = stationName;
		mmp[id].second = t;
	}

	void checkOut(int id, string stationName, int t) {
		int time = t - mmp[id].second;
		string startStation = mmp[id].first;
		mp[startStation][stationName].first += time;
		mp[startStation][stationName].second++;
	}

	double getAverageTime(string startStation, string endStation) {
		auto c = mp[startStation][endStation];
		return c.first * 1.0 / c.second;
	}
};
```



## 找到所有好字符串

给你两个长度为 `n` 的字符串 `s1` 和 `s2` ，以及一个字符串 `evil` 。请你返回 **好字符串** 的数目。

**好字符串** 的定义为：它的长度为 `n` ，字典序大于等于 `s1` ，字典序小于等于 `s2` ，且不包含 `evil` 为子字符串。

由于答案可能很大，请你返回答案对 10^9 + 7 取余的结果。

 

**示例 1：**

```
输入：n = 2, s1 = "aa", s2 = "da", evil = "b"
输出：51 
解释：总共有 25 个以 'a' 开头的好字符串："aa"，"ac"，"ad"，...，"az"。还有 25 个以 'c' 开头的好字符串："ca"，"cc"，"cd"，...，"cz"。最后，还有一个以 'd' 开头的好字符串："da"。
```

**示例 2：**

```
输入：n = 8, s1 = "leetcode", s2 = "leetgoes", evil = "leet"
输出：0 
解释：所有字典序大于等于 s1 且小于等于 s2 的字符串都以 evil 字符串 "leet" 开头。所以没有好字符串。
```

**示例 3：**

```
输入：n = 2, s1 = "gx", s2 = "gz", evil = "x"
输出：2
```

 

**提示：**

- `s1.length == n`
- `s2.length == n`
- `s1 <= s2`
- `1 <= n <= 500`
- `1 <= evil.length <= 50`
- 所有字符串都只包含小写英文字母。

解答：

数位dp + KMP 问题，可以直接参照[官方题解](https://leetcode-cn.com/problems/find-all-good-strings/solution/zhao-dao-suo-you-hao-zi-fu-chuan-by-leetcode-solut/)。

