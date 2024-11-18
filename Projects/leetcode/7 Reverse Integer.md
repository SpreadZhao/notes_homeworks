---
num: "7"
title: "Reverse Integer"
link: "https://leetcode.cn/problems/reverse-integer/description/"
tags:
  - leetcode/difficulty/medium
---

# 题解

讲真，这题是数学题。其实如果不考虑int溢出，很简单，直接取最后一位，然后不断\*10累加就行了：

```cpp
int Solution::reverse(int x) {
    int reversed = 0;
    int xx = x;
    while (xx != 0) {
        const int curr = xx % 10;
        reversed = reversed * 10 + curr;
        xx /= 10;
    }
    return reversed;
}
```

这里有一点，我们连符号都不用考虑，因为负数进行这样计算，也是负数的累加。

但是，如果涉及到溢出，同时还不让你用long，那就不好办了。思路就是利用数学推倒。

核心思想是，我们要提前判断出来`reversed * 10 + curr`的值和`INT_MAX`以及`INT_MIN`的大小关系。并且不能对这个表达式进行计算。那么很显然，就想到了把`* 10 + curr`给挪到另一边去。

以正数部分为例。假设我们这个表达式写成这样：

$$
rev_A = rev_B \times 10 + curr
$$

然后我们要判断的是：

$$
rev_A > 2^{31} - 1 = 2147483647 = INT\_MAX
$$

就只需要判断：

$$
\begin{array}{l}
& rev_B \times 10 + curr > INT\_MAX \\
\Rightarrow & rev_B > \dfrac{INT\_MAX - curr}{10} \\
& 其中，curr \in [0, 9]
\end{array}
$$

带入$INT\_MAX$的值，其实就可以知道，我们需要让：

$$
rev_B > [214748363, 214748364]
$$

那么显然，如果$rev_B > 214748364$的话，结果一定是溢出的（也可以带入一下，比如214748365看看）。

但是，还有个特殊情况，就是恰好等于214748364。此时单独计算，发现乘以10之后是2147483640，因此，如果curr此时还大于7的话，最后的结果还是溢出的。

所以，在每一次循环中，对正数的判断条件：

```cpp
// BOUNDARY_P = INT_MAX / 10
if (reversed > BOUNDARY_P || reversed == BOUNDARY_P && curr > 7) {
	return 0;
}
```

接下来看负数，使用一样的方法，但是注意此时$curr \in [-9, 0]$。得到最后应该让：

$$
rev_B < [-214748364, -214748363]
$$

所以，只要$rev_B < -214748364$，结果也是溢出的。然后还是等于的情况。带入，乘以10之后就是-2147483640。因此如果此时$curr < -8$，最后的结果就是溢出的。因此负数的判断是：

```cpp
// BOUNDARY_N = INT_MIN / 10
if (reversed < BOUNDARY_N || reversed == BOUNDARY_N && curr < -8) {
	return 0;
}
```

最终代码：

```cpp
int Solution::reverse(int x) {
    int reversed = 0;
    int xx = x;
    constexpr int BOUNDARY_P = INT_MAX / 10;
    constexpr int BOUNDARY_N = INT_MIN / 10;
    while (xx != 0) {
        const int curr = xx % 10;
        if (reversed > BOUNDARY_P || reversed == BOUNDARY_P && curr > 7) return 0;
        if (reversed < BOUNDARY_N || reversed == BOUNDARY_N && curr < -8) return 0;
        reversed = reversed * 10 + curr;
        xx /= 10;
    }
    return reversed;
}
```

> [!attention]
> 这里其实边界的判断我没想出来，是题解里的。