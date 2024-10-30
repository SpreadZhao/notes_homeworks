---
num: "2405"
title: "Optimal Partition of String"
link: "https://leetcode.cn/problems/optimal-partition-of-string/description/"
tags:
  - leetcode/difficulty/medium
---

# 题解

没想到，这题居然做的这么顺。

很简单，每次遍历一个字母，看它是否存在。如果不存在，就存着，如果存在，那么计数+1，然后存储清零重新来。最后的结果就是在此基础上再+1，因为最后段我们没有+1就出循环了。

至于存储是用数组还是哈希表，随便。我用的是数组：

```cpp
int Solution::partitionString(string s) {
    bool appear[26] = {};
    int count = 0;
    for (int i = 0; i < s.length(); i++) {
        const char ch = s[i];
        if (appear[ch - 'a']) {
            count++;
            memset(appear, 0, sizeof(appear));
        }
        appear[ch - 'a'] = true;
    }
    return ++count;
}
```

另外，题解里还有一种更巧的解法，用位运算。

![[Projects/leetcode/resources/Drawing 2024-10-31 01.12.30.excalidraw.svg]]

让0代表没出现，1代表出现，这样一共26bit就能表示所有的情况。比数组更简单。

写起来也很简单：

```cpp
int Solution::partitionString2(string s) {
    int count = 0, merge = 0;
    for (const auto ch : s) {
        if ((merge & (1 << ch - 'a')) != 0) {
            count++;
            merge = 0;
        }
        merge |= (1 << ch - 'a');
    }
    return ++count;
}
```

一模一样的思路，就是存储出现过的从数组变成了一个int。