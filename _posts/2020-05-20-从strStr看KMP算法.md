---
layout:     post
title:      从strStr看KMP算法
subtitle:   Leetcode题解系列
date:       2020-05-16
author:     YunLambert
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - Algorithm

---

> 从strStr看KMP算法

Leetcode上的一道easy题：[https://leetcode-cn.com/problems/implement-strstr/](https://leetcode-cn.com/problems/implement-strstr/)

暴力 + 剪枝确实能过，但是KMP确实能够简化很多。

先放上暴力+剪枝的做法：

```C++
class Solution {
public:
    int strStr(string haystack, string needle) {
        if (needle == "") return 0;
        int i;
        bool flag = false;
        for (i = 0; i < haystack.size(); ++i) {
            if (haystack.size() - i + 1 < needle.size()) break;  // 剪枝
            int t = i, j = 0;
            while (j < needle.size() && t < haystack.size() && haystack[t] == needle[j]) {
                t++, j++;
            }
            if (j == needle.size()) {
                flag = true;
                break;
            }
        }
        if (flag) return i;
        return -1;
    }
};
```



