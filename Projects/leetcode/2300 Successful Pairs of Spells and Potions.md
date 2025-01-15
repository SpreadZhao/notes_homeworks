---
num: "2300"
title: "Successful Pairs of Spells and Potions"
link: "https://leetcode.cn/problems/successful-pairs-of-spells-and-potions/description/"
tags:
  - leetcode/difficulty/medium
---

# 题解

这道题感觉是我最傻逼的一题，写了三个小时，最后写了个四不像方法。。。

首先，这道题其实如果不带脑子做，其实非常简单，只要乘起来满足条件就记一下数就行了，直接给代码：

```cpp
vector<int> Solution::successfulPairs(vector<int> &spells, vector<int> &potions, long long success) {
    vector<int> res;
    for (auto spell: spells) {
        int count = 0;
        for (auto potion: potions) {
            long long product = static_cast<long long>(spell) * potion;
            if (product >= success) {
                ++count;
            }
        }
        res.push_back(count);
    }
    return res;
}
```

但是显然，这样做肯定是不行的。时间复杂度不符合要求。所以需要改进。

改进的思路我瞄了一眼之前做的才知道，当然也没全喵，只看到了一个排序。看到这个一下就想到了。因为题目里只要求输出数量，所以，其实理论上，只要我们找到了按顺序的，第一个乘起来正好>= success的元素，那它和它后面所有元素的数量就是答案了。实现也很简单，直接给代码：

```cpp
vector<int> Solution::successfulPairs2(vector<int> &spells, vector<int> &potions, long long success) {
    vector<int> res;
    sort(potions.begin(), potions.end());
    for (auto spell: spells) {
        int count = 0;
        for (int i = 0; i < potions.size(); i++) {
            int potion = potions[i];
            long long product = static_cast<long long>(spell) * potion;
            if (product >= success) {
                count += potions.size() - i;
                break;
            }
        }
        res.push_back(count);
    }
    return res;
}
```

emm，还是超时了。。。所以，感觉这里还需要优化。又看了眼之前做的，才发现，我们需要用二分查找实现这个思路。也就是，用二分查找找到第一个满足条件的元素的要求。

我最后写的版本稍微变了一下，找的是乘起来< success的最后一个元素。

但是显然，我不太擅长二分查找，因为总是把控不好循环跳出之后到底是什么情况。所以做了非常多的额外条件判断。下面是代码：

```cpp
vector<int> Solution::successfulPairs3(vector<int> &spells, vector<int> &potions, long long success) {
    vector<int> res;
    sort(potions.begin(), potions.end());
    for (auto spell: spells) {
        int low = 0, high = potions.size() - 1;
        bool set = false;
        while (low <= high) {
            int mid = (low + high) / 2;
            if (low == high && (mid == 0 || mid == potions.size() - 1)) {
                break;
            }
            long long thisProduct = static_cast<long long>(spell) * potions[mid];
            long long nextProduct = static_cast<long long>(spell) * potions[mid + 1];
            if (thisProduct < success) {
                if (nextProduct >= success) {
                    res.push_back(potions.size() - mid - 1);
                    set = true;
                    break;
                }
                low = mid + 1;
            } else {
                high = mid - 1;
            }
        }
        if (!set) {
            if (low == high) {
                long long product = static_cast<long long>(spell) * potions[low];
                if (product >= success) {
                    res.push_back(potions.size() - low);
                } else {
                    res.push_back(potions.size() - low - 1);
                }
            } else {
                // low > high
                if (high < 0) {
                    high = 0;
                }
                if (low > potions.size() - 1) {
                    high = potions.size() - 1;
                }
                long long product = static_cast<long long>(spell) * potions[high];
                if (product >= success) {
                    res.push_back(potions.size() - high);
                } else {
                    res.push_back(potions.size() - high - 1);
                }
            }
        }
    }
    return res;
}
```

好大一拖沟施代码。说实话我都不想解释，因为本质上还是为了实现找到第一个满足条件的元素而已。只是因为情况太多，比如所有的元素都满足条件、都不满足条件，有多个相等的元素恰好满足条件等等。。。

显然，这段代码是一次次试错产生的。就像有很多补丁一样。这样的代码是绝对要避免的，因而我说这是一坨大狗屎代码。

- [ ] #TODO tasktodo1736008827362 1. 看看题解里究竟是怎么进行二分查找的；2. 尝试优化这坨大狗屎。 ➕ 2025-01-05 ⏫ 🆔 9vo8gi 

# 遗漏的case

