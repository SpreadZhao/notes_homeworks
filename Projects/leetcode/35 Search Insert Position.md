---
num: "35"
title: "Search Insert Position"
link: "https://leetcode.cn/problems/search-insert-position/description/"
tags:
  - leetcode/difficulty/easy
---

# 题解

既然要$O(log\ n)$时间复杂度，那肯定就是二分查找了。这里和二分查找唯一的区别是，如果没找到，需要看看具体的情况：

```cpp
int Solution::searchInsert(vector<int> &nums, int target) {
    int i = 0, j = nums.size() - 1;
    while (i < j) {
        const int mid = (i + j) / 2;
        const int midValue = nums[mid];
        if (midValue == target) {
            return mid;
        }
        if (midValue < target) {
            // right part of
            i = mid + 1;
        } else if (midValue > target) {
            j = mid - 1;
        }
    }
    if (nums[i] < target) {
        return i + 1;
    }
    // equal or larger
    return i;
}
```

退出循环后，`i == j`一定成立。因此，二者指向的值要么比target大，要么比target小，要么和target相等。之所以还有可能相等，是因为退出循环那一次才找到，是进不了while循环里的`midValue == target`的。

所以分开讨论：

1. 如果`nums[i] < target`，那么target插入后应该在`nums[i]`后面，那它的编号就是`i + 1`；
2. 如果`nums[i] == target`，那么target本身就存在，返回i；
3. 如果`nums[i] > target`，那么target插入后应该代替了`nums[i]`的位置，它的编号还是`i`。