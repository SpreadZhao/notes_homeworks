---
num: "129"
title: "Sum Root to Leaf Numbers"
link: "https://leetcode.cn/problems/sum-root-to-leaf-numbers/description/"
tags:
  - leetcode/difficulty/medium
---

# 题解

这题属于灵感迸发，之前卡了挺久，突然开窍了。

我们就用这个举例子：

![[Projects/leetcode/resources/Pasted image 20250316164100.png]]

答案应该该是495 + 491 + 40 = 1026。

一遇到树，那一定是要遍历树的。那么我们就要在遍历的时候，顺便把这个值给算出来。

首先观察，这棵树有三个叶子节点，答案也由三个数字组成。其实不难看出，这题就是每个叶子节点代表一个数字，然后把它们加起来。

所以，这题的核心逻辑就是遍历到每一个叶子节点，遍历到的时候顺便把数字带上。最后把它们都加起来就行了。

那么，问题就在于，数字是怎么算出来的。

也很好观察，比如从4往下遍历，到了9，那么很容易得出新的值是$4 \times 10 + 9 = 49$。然后再遍历到5，就能得到最后的值是$49 \times 10 + 5 = 495$。

从这里就能大概构想出这个递归算数字的方式，也就是一个数字不断往子递归去传，然后在子递归中更新。最后到了叶子节点的时候，最后再算一次，然后直接返回结果。

思路很好想，但是怎么设计代码呢？我们来模拟一下。比如遍历到了9这个节点的时候，此时的数字算出来就应该是49。那么这个49往上提交的答案应该是多少？显然，是从9往下面走的所有数字之和。在这个例子里就应该是495 + 491。

因此，我们把49再次传递给左右子递归，然后把二者的返回值加起来，就能得到我们想要的495 + 491：

```cpp
static int addThis(int sum, TreeNode *curr) {
    ... ...
    // sum == 49
    int res = 0;
    if (curr->left != nullptr) {
        res += addThis(sum, curr->left);    // res += 495
    }
    if (curr->right != nullptr) {
        res += addThis(sum, curr->right);   // res += 491
    }
    return res;
}
```

同样，我们把这套逻辑换到最一开始的节点4上，会发现：

```cpp
static int addThis(int sum, TreeNode *curr) {
    ... ...
    // sum == 4
    int res = 0;
    if (curr->left != nullptr) {
        res += addThis(sum, curr->left);    // res += (495 + 491)
    }
    if (curr->right != nullptr) {
        res += addThis(sum, curr->right);   // res += 40
    }
    return res;
}
```

这里对left的子递归，就是上面从49传递下去最后的结果，即495 + 491；而对于right的子递归显然就是数字40。因此最后得到了答案。

前面省略的部分，就是处理叶子节点的过程。全部代码：

```cpp
static int addThis(int sum, TreeNode *curr) {
    if (curr == nullptr) {
        // In case that root is null
        return sum;
    }
    sum = sum * 10 + curr->val;
    if (curr->left == nullptr && curr->right == nullptr) {
        return sum;
    }
    int res = 0;
    if (curr->left != nullptr) {
        res += addThis(sum, curr->left);
    }
    if (curr->right != nullptr) {
        res += addThis(sum, curr->right);
    }
    return res;
}

int Solution::sumNumbers(TreeNode *root) {
    return addThis(0, root);
}
```

- [ ] #TODO tasktodo1742115508539 广度优先搜索的解法 [129. 求根节点到叶节点数字之和 - 力扣（LeetCode）](https://leetcode.cn/problems/sum-root-to-leaf-numbers/solutions/464666/qiu-gen-dao-xie-zi-jie-dian-shu-zi-zhi-he-by-leetc/)。 ➕ 2025-03-16 🔽 🆔 5burzz 