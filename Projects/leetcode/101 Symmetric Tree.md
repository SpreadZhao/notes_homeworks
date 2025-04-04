---
num: "101"
title:
  - Symmetric Tree
link: https://leetcode.cn/problems/symmetric-tree/description/
tags:
  - leetcode/difficulty/easy
---

# 题解

这题算是全靠自己做出来的，但是还是想了有点久。一个easy这么墨迹，有点废物了属于是。

第一眼想到的，是层序遍历，然后从中间对半匹开看一不一样，但很显然这是一个很蠢的做法。

一定是在遍历的过程中，就能确定下来。

然后我就想到了，以先序遍历举例子，我们的路径是“根-左-右”。我们记一下这个节点遍历的路径，就不难发现，如果我们再走一遍“根-右-左”的遍历，正好每次走到的节点，都是“根-左-右”遍历的镜像节点。

因此，这里我们也是先序遍历，但是每次要挪两个节点。一个遵循“根-左-右”，另一个遵循“根-右-左”。这俩节点要么都为空，要么取值必须相等。剩下的情况都是错的。所以，这次我们递归遍历的函数应该有两个参数，一个是“根-左-右”的curr，另一个是“根-右-左”的curr：

```cpp
static bool traverse(TreeNode *curr1, TreeNode *curr2);
```

在`traverse`里面，比较一下上面说的事情就行了。当然，这里有很多边界case要注意，我做题的时候就没考虑全，必须把一个是空、两个都是空、两个都不是空的情况都考虑到才行。下面是完整代码：

```cpp
static bool traverse(TreeNode *curr1, TreeNode *curr2) {
    if (curr1 == nullptr && curr2 == nullptr) {
        return true;
    }
    if (curr1 != nullptr && curr2 == nullptr) {
        return false;
    }
    if (curr1 == nullptr) {
        return false;
    }
    if (curr1->val != curr2->val) {
        return false;
    }
    return traverse(curr1->left, curr2->right) && traverse(curr1->right, curr2->left);
}

bool Solution::isSymmetric(TreeNode *root) {
    if (root == nullptr) {
        return true;
    }
    if (root->left == nullptr && root->right == nullptr) {
        return true;
    }
    if (root->left != nullptr && root->right == nullptr) {
        return false;
    }
    if (root->left == nullptr) {
        return false;
    }
    return traverse(root->left, root->right);
}
```