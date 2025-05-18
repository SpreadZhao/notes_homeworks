---
num: "2316"
title: Count Unreachable Pairs of Nodes in an Undirected Graph
link: https://leetcode.cn/problems/count-unreachable-pairs-of-nodes-in-an-undirected-graph/
tags:
  - leetcode/difficulty/medium
  - algo/depth-first-search
  - algo/breadth-first-search
  - algo/union-find
  - algo/graph
---

# 题解

## 我的解法

这道题我想可以直接说出思路。看例子：

![[Projects/leetcode/resources/Pasted image 20250518181229.png|400]]

这里结果让我们输出14，也就是**每一个孤岛中的节点两两组合**。那显然我们能得出解决思路：

- 算出每一个孤岛中的节点数量，这里是4, 1, 2。
- 然后从第一个开始，往后面一直求和。说不太懂，直接写出来就是$4 \times 1 + 4 \times 2 + 1 \times 2 = 14$。

所以，最大的问题就是解出每个孤岛中节点的数量。在[[Projects/leetcode/1319 Number of Operations to Make Network Connected|1319 Number of Operations to Make Network Connected]]中，我们已经能算出这样一个组合中有多少个孤岛了，把那个问题看一遍，然后，只需要在dfs的时候统计一下孤岛里的节点就行了。

首先，先看一遍1319的实现方式，然后接着看。我们看这段代码：

```cpp
// dfs from 0 to n-1
for (int i = 0; i < n; i++) {
	if (!visited[i]) {
		response.islandNum++;
		dfsCore(i, map, visited);
	}
}
```

这段代码是在统计每一个岛屿的数量。**只要在岛屿里还能发现新的节点，这一层`dfsCore()`就不会返回**。所以，每次`dfsCore()`返回之后，代表从这个节点出发进行dfs，所有的节点都已经被访问了。此时需要增加岛屿的数量。

因此，我们可以发现，**所有在`dfsCore()`中发现的节点，都来自于同一个岛屿**。

所以，改造代码如下：

```cpp
// dfs from 0 to n-1
for (int i = 0; i < n; i++) {
	if (!visited[i]) {
		response.islandNum++;
		long long nodeCount = 0;
		dfsCore(i, map, visited, nodeCount);   // update node count in one island
		islandSizes.emplace_back(nodeCount);
	}
}
```

我们在每发现一个新岛屿的时候，传入一个节点数量，然后在里面更新这个数量。最后，当`dfsCore()`返回，`nodeCount`就应该是这个岛屿的节点数量，将它更新进去就行了。

至于`nodeCount`的更新，也很简单，在每一个节点设置`visited`的时候，加一下就行了：

```cpp
void dfsCore(int start, const map<int, vector<int> > &edgesMap, vector<bool> &visited, long long &currIslandNodeCount) {
    if (visited[start]) {
        return;
    }
    visited[start] = true;
    currIslandNodeCount++;   // add node count in current island
    if (!edgesMap.count(start)) {
        return;
    }
    for (const auto neighbor: edgesMap.at(start)) {
        if (!visited[neighbor]) {
            dfsCore(neighbor, edgesMap, visited, currIslandNodeCount);
        }
    }
}
```

这样，我们就能得到这个最终的节点数量数组，拿上面的例子来说，就是`[4, 1, 2]`（顺序不重要）。下一步，就是把14算出来。

我们很容易写出这样的代码：

```cpp
long long result = 0;
for (int i = 0; i < islandSizes.size() - 1; i++) {
	for (int j = i + 1; j < islandSizes.size(); j++) {
		result += islandSizes[i] * islandSizes[j];
	}
}
return result;
```

但是，超时了。。。所以，得想一个更快的算法，计算一个数组中两两元素之和。注意，如果数组里只有一个元素，应该返回0。

我问了AI，它给出了这个办法。

我们要算的数组记为$S$。那么我们要算的其实是：

$$
\sum\limits_{0 \leq i < j < n} S_i \times S_j
$$

**注意到**（南泵，AI的注意力惊人）：

$$
\sum\limits_{0 \leq i < j < n} S_i \times S_j = \dfrac{1}{2} \left( \left( \sum\limits_{i = 0}^{n - 1} S[i] \right)^2 - \sum\limits_{i = 0}^{n - 1} S[i]^2 \right)
$$

说人话，其实就是：

$$
\dfrac{和的平方 - 平方的和}{2}
$$

就拿`[4, 1, 2]`举例子。和的平方是$(4 + 1 + 2)^2 = 49$，平方的和是$4^2 + 1^2 + 2^2 = 21$。这样一算结果是$\dfrac{49 - 21}{2} = 14$。就是最终结果。很神奇。不过我依然不知道这个是怎么推导出来的。。

总之，最后有了这样一个工具函数：

```cpp
long long CommonUtil::quickPairSum(const vector<long long>& nums) {
    long long sum = 0;
    long long squaredSum = 0;
    for (const long long num : nums) {
        sum += num;
        squaredSum += num * num;
    }
    return (sum * sum - squaredSum) / 2;
}
```

这样，我们就把时间复杂度从$O(n^2)$缩到了$O(n)$。最后也过了。

- [ ] #TODO tasktodo1747570120612 补充官方两个解法 ➕ 2025-05-18 ⏫ 🆔 mwsdmi

## Leetcode DFS

## Leetcode Union Find

