---
num: "46"
title: "Permutations"
link: "https://leetcode.cn/problems/permutations/description/"
tags:
  - leetcode/difficulty/medium
---

# 题解

这题和[[Projects/leetcode/17 Letter Combinations of a Phone Number|17 Letter Combinations of a Phone Number]]很像。都是给出不同的组合。当然解法也是一样的，画出这棵树，就有思路了。比如1 2 3 4，那么这棵树就是这样的：

![[Projects/leetcode/resources/Drawing 2024-11-16 18.44.08.excalidraw.svg]]

每次选一个就行了。就像我们拼接string的做法一样，这里拼接的是int。给出递归的参数：

- `result`：最后的结果，`vector<vector<int>>`
- `curr`：已经拼上的数字
- `nums`：题里给的组合
- `num`：当前要拼的数字

之所以要把`num`暴露到递归方法的参数上，是为了在第一层递归开始的时候就能逐个确定。和dfs一样，我们不是也要在递归一开始的时候就visit每一个节点吗？假设我们的递归函数叫pickOne，那么我们可以这么调用：

```cpp
for (auto num : nums) {
	pickOne(result, curr, nums, num);
}
```

然后递归函数里面注意的：

1. 先把num放到curr里；
2. 如果放入后，curr的size和nums一样了，就代表选完了，所以这层递归可以退出了；
3. 遍历nums，再次找到一个没访问过的数字，进行递归。这次递归就是树形结构伸展的重要途径，当一次return之后，会跳出里面的层级，找到一个新数字（在当前层级），并开始新的递归。这里结合dfs的实现，或者[[Projects/leetcode/17 Letter Combinations of a Phone Number|17 Letter Combinations of a Phone Number]]都能比较好地理解。

代码：

```cpp
void pickOne(vector<vector<int>> &result, vector<int> curr, vector<int> &nums, int num) {
    if (!contains(curr, num)) {
        curr.push_back(num);
    }
    if (curr.size() == nums.size()) {
        result.push_back(curr);
        return;
    }
    for (auto n : nums) {
        if (!contains(curr, n)) {
            pickOne(result, curr, nums, n);
        }
    }
}

vector<vector<int>> Solution::permute(vector<int> &nums) {
    vector<vector<int>> result;
    vector<int> curr;
    for (auto num : nums) {
        pickOne(result, curr, nums, num);
    }
    return result;
}
```

> [!note]-
> 这里的`contains`是我自己写的函数：
> 
> ~~~cpp
> bool contains(vector<int> &nums, int target) {
>     const auto res = find(nums.begin(), nums.end(), target);
>     return res != nums.end();
> }
> ~~~

这里出现了一个插曲。就是一开始我的递归函数里的curr用的是引用。结果很扯：

```
[1, 2, 3]
[1, 2, 3]
[1, 2, 3]
```

这里主要的原因就是，curr这种临时变量和string一样，在传到“下层递归”的时候，一定要是定值。所以你如果非要引用的话，就需要在当前层级的递归已经确保没问题之后，将元素给移除。

- [ ] #TODO tasktodo1731754872199 这题能不用递归做吗？顺便看看lc上的解法。 ➕ 2024-11-16 ⏫ 🆔 xs5qif 