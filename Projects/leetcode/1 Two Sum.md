---
num: "1"
title: "Two Sum"
link: "https://leetcode.cn/problems/two-sum/description/"
tags:
  - leetcode/difficulty/easy
---

# 题解

lc的第一题。这个很经典了，直接暴力遍历的复杂度是$O(n^2)$，所以我们肯定要更优解。

如果数组原本就是有序的，我们显然可以用双指针来做，往中间靠就行。但是，这道题要求我们返回的是index，同时输入数组是无序的。因此在排序之后index会被打乱。那么怎么搞？显然我们需要记录一下。这里的做法是将每个元素扩展一下，再记录一下原来的index：

```cpp
vector<vector<int>> nums2 = {};
for (int i = 0; i < nums.size(); i++) {
	nums2.push_back({nums[i], i});
}
sort(nums2.begin(), nums2.end(),  [](const vector<int> &a, const vector<int> &b) {
	return a[0] < b[0];
});
```

这样，nums2里的每一个元素，0号就是数字，1号是原来的index。这样排序之后也能记住了。后面正常用双指针来做：

```cpp
vector<int> Solution::twoSum(vector<int> &nums, int target) {
    vector<vector<int>> nums2 = {};
    for (int i = 0; i < nums.size(); i++) {
        nums2.push_back({nums[i], i});
    }
    sort(nums2.begin(), nums2.end(),  [](const vector<int> &a, const vector<int> &b) {
        return a[0] < b[0];
    });
    int i = 0, j = nums.size() - 1;
    while (i < j) {
        const int sum = nums2[i][0] + nums2[j][0];
        if (sum == target) {
            return {nums2[i][1], nums2[j][1]};
        }
        if (sum > target) j--;
        else i++;
    }
    return {};
}
```

复杂度显然是快排+一次遍历，$O(nlogn) + O(n) = O(nlogn)$。

那么有没有更好的办法？其实，求计算结果等于某个值这种，都可以反过来考虑：**如果我固定一个元素a，那么其实就是在数组中剩下的元素里找target - a这个元素**。

所以，伪代码应该是这样的：

```cpp
for (auto num : nums) {
	if (other = find(nums, target - num)) {
		return {num.index, other.index};
	}
}
```

显然，这里有一些问题要解决：

1. find函数应该怎么实现？肯定不能用for循环一个个找，不然又变成$O(n^2)$了；
2. 这里index又怎么才能获取到？

先看第一个问题，我们举个例子：

```
[2,7,11,15]
```

当target = 9，num = 2的时候，我们其实希望找7对吧。但是我们又不能用for循环一个个遍历找到7，那样太慢了。那有没有别的方式呢？答案是，用哈希表。

当我们用2找的时候，把2存下来。**这样，当我们再用7找的时候，就能找到哈希表里的2了**。因为题目里说了，每一个输入的答案是唯一的，因此如果对于一个num，它是答案中的一员的话，那么另一员其实也是一定的，并且它俩都能互相找出来。

所以，我们将这题拆成两部分：第一部分是存哈希表，第二部分是通过哈希表找到另一半。

那么find的实现也清楚了：直接从hash表里找target - num。如果没找到，那只有两种情况：要么num就不是答案中的一个，要么是答案中的一个，但是另一半还没存到表里。所以，此时我们只需要把num存到hash表里，这样通过另一半来找的时候就能找到了。

接下来是第二个问题，index怎么确定。这里我们注意下，这个hash表我们只用了key，value还没用上。显然，value很适合用来存index。这样我们在hash表中找到另一半的时候，另一半的value就是它的index。

```cpp
vector<int> Solution::twoSum2(vector<int> &nums, int target) {
    unordered_map<int, int> numToIndex;
    for (int i = 0; i < nums.size(); i++) {
        auto it = numToIndex.find(target - nums[i]);
        if (it != numToIndex.end()) {
            return {it->second, i};
        }
        numToIndex[nums[i]] = i;
    }
    return {};
}
```

因为hash表find的复杂度就是$O(1)$，所以整体复杂度降到了$O(n)$。
