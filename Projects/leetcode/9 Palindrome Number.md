---
num: "9"
title: Palindrome Number
link: https://leetcode.cn/problems/palindrome-number/description/
tags:
  - leetcode/difficulty/easy
  - algo/math
---

# 题解

这个没啥好说的。第一眼想到的肯定是转string+双指针：

```cpp
bool Solution::isPalindrome(int x) {
    if (x < 0) {
        return false;
    }
    if (x == 0) {
        return true;
    }
    string s = to_string(x);
    int i = 0, j = s.length() - 1;
    while (i <= j) {
        if (i == j) {
            return true;
        }
        if (s[i] != s[j]) {
            return false;
        }
        i++;
        j--;
    }
    return true;
}
```

然后，又问我们能不能不转成string解决。显然，方法就是将这个数字倒转过来，如果一样那就是回文。那重要的就是倒转的过程。

我们只需要每次把这一位取出来，然后拼在原来的后面，就成了：

```cpp
while (xx > 0) {
	const int curr = xx % 10;
	y = y * 10 + curr;
	xx /= 10;
}
```

这样，y就是倒转后的x。比较就行了：

```cpp
bool Solution::isPalindrome2(int x) {
    if (x < 0) {
        return false;
    }
    if (x == 0) {
        return true;
    }
    long y = 0;
    int xx = x;
    while (xx > 0) {
        const int curr = xx % 10;
        y = y * 10 + curr;
        xx /= 10;
    }
    return y == x;
}
```

# 遗漏的case

一开始y是int，没考虑到溢出的问题。因为倒转之后y有可能超过int限制，所以用long。