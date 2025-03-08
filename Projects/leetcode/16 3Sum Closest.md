---
num: "16"
title: "3Sum Closest"
link: "https://leetcode.cn/problems/3sum-closest/description/"
tags:
  - leetcode/difficulty/medium
---

# 题解

这题又是上一道[[Projects/leetcode/15 3 Sum|15 3 Sum]]的升级版。判断的不是和等于0，而是和target最接近的一次。

解法和上一道一模一样。还是固定i，j和k走双指针。区别就是，增加了距离的判断：

- 如果和正好是target，那么直接返回；
- 如果和< target，本来要让j++，这里多算一下diff；
- 如果和> target，本来要让k--，这里多算一下diff。

```cpp
int Solution::threeSumCloset(vector<int> &nums, int target) {
    sort(nums.begin(), nums.end());
    int diff = INT_MAX;
    int res = INT_MAX;
    for (int i = 0; i < nums.size(); i++) {
        if (i > 0 && nums[i] == nums[i - 1]) {
            continue;
        }
        int j = i + 1, k = nums.size() - 1;
        while (j < k) {
            const int sum = nums[i] + nums[j] + nums[k];
            if (sum == target) {
                return sum;
            } else if (sum < target) {
                if (abs(sum - target) < diff) {
                    diff = abs(sum - target);
                    res = sum;
                }
                j++;
            } else {
                if (abs(sum - target) < diff) {
                    diff = abs(sum - target);
                    res = sum;
                }
                k--;
            }
        }
    }
    return res;
}
```

# 遗漏的case

