---
num: "540"
title: "Single Element in a Sorted Array"
link: "https://leetcode.cn/problems/single-element-in-a-sorted-array/description/"
tags:
  - leetcode/difficulty/medium
---

# 题解

这题我一看到$O(log\ n)$，就马上反应过来应该是用二分查找。但是具体二分，咋想没想出来。这个题的关键是，当你通过low, high确定了mid之后，要能判断出目标的数字是在左半部分还是右半部分才行。但是具体怎么判断一直没想出来。直到瞥了一眼代码，一下就想到了，确实很巧。

对于我们第一次确定的mid，`nums[mid]`一定只有两种可能：

1. 它就是我们要找的数字，那么它一定**和前面，后面都不相等**；
2. 它不是我们要找的数字，那么它一定**要么和前面相等，要么和后面相等**。

第一种就不说了，直接返回。那么第二种情况下，其实我们可以做一个假设。

假设`nums[mid]`和前面的元素相等。那么如果左半部分全部都是正常的数字的话，**从low到mid的所有数字，一定是偶数个**。因为都正常，它们都会出现两次。

其实懂了这个点，这道题就做出来了，后面就是根据不同的情况分类讨论。一共四种情况：

1. 和前面的相等 - 左半部分是偶数个 -> target在右半部分；
2. 和前面的相等 - 左半部分是奇数个 -> target在左半部分；
3. 和后面的相等 - 左半部分是偶数个 -> target在左半部分；
4. 和后面的相等 - 左半部分是奇数个 -> target在右半部分。

最后，只需要注意一下**【个数】和【从0开始的下标】的区别，以及左半，右半部分是否应该包含`nums[mid]`**，这道题就解决了。直接给代码：

```cpp
int Solution::singleNonDuplicate(vector<int> &nums) {
    int low = 0, high = nums.size() - 1;
    while (low <= high) {
        int mid = (low + high) / 2;
        if (mid == 0 || mid == nums.size() - 1) {
            // overflow
            return nums[mid];
        }
        if (nums[mid] == nums[mid - 1]) {
            if (mid % 2 == 0) {
                // target is in low - mid
                high = mid;
            } else {
                // target is in (mid + 1) - high
                low = mid + 1;
            }
        } else if (nums[mid] == nums[mid + 1]) {
            if (mid % 2 == 0) {
                // target is in mid - high
                low = mid;
            } else {
                // target is in low - (mid - 1)
                high = mid - 1;
            }
        } else {
            return nums[mid];
        }
    }
    return nums[low];
}
```

# 遗漏的case

上面提交的时候，我没有加这段代码：

```cpp
if (mid == 0 || mid == nums.size() - 1) {
	// overflow
	return nums[mid];
}

// 也可以这么写
if (low == high) {
	// overflow
	return nums[mid];
}
```

这会导致overflow，但是本地的编译器很智能，把这个错误给隐蔽了。如果输入是`[1, 2, 2, 3, 3]`这种情况，那最终会定位到`mid = 0`，此时+1和-1都会导致overflow。要处理。处理也很简单，因为已经到了最边上，并且其实此时也可以认为low == high，所以直接返回就行了。

- [ ] #TODO tasktodo1735059780021 这题有个更高级的做法：[Single Element in a Sorted Array - LeetCode](https://leetcode.com/problems/single-element-in-a-sorted-array/solutions/3212178/day-52-binary-search-easiest-beginner-friendly-sol/)。 ➕ 2024-12-25 🔼 🆔 ugrrnz 