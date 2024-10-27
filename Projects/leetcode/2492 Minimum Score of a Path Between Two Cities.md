---
num: "2492"
title:
  - Minimum Score of a Path Between Two Cities
link: https://leetcode.cn/problems/minimum-score-of-a-path-between-two-cities/description/
tags:
  - leetcode/difficulty/medium
---

# 题解

说的是能从1到达n的所有路径里最短的那个，那么显然，dfs，然后找出最短的那条就成了。

也就是一个dfs加上记忆的过程。

首先，依然是从输入构建邻接表：

```cpp
map<int, vector<vector<int>>> edgeMap;
for (auto road: roads) {
	const int node1 = road[0];
	const int node2 = road[1];
	const int distance = road[2];
	auto neighbor1 = vector<int>{node2, distance};
	auto neighbor2 = vector<int>{node1, distance};
	if (!edgeMap.count(node1)) {
		auto neighbors = vector<vector<int>>();
		neighbors.emplace_back(neighbor1);
		edgeMap[node1] = neighbors;
	} else {
		edgeMap[node1].emplace_back(neighbor1);
	}
	if (!edgeMap.count(node2)) {
		auto neighbors = vector<vector<int>>();
		neighbors.emplace_back(neighbor2);
		edgeMap[node2] = neighbors;
	} else {
		edgeMap[node2].emplace_back(neighbor2);
	}
}
```

这没什么好说的，这样我们通过map，能查到一个`vector<vector<int>>`，每一个元素是一个大小为2的数组，包含邻居节点和路径的距离。

接下来，按照dfs的套路来走就行了：

```cpp
static void dfs_minScore(int &minDis, vector<bool> &visited, int curr, map<int, vector<vector<int>>> edgeMap) {
    visited[curr - 1] = true;
    vector<vector<int>> neighbors = edgeMap[curr];
    for (auto neighbor : neighbors) {
        auto other = neighbor[0];
        auto distance = neighbor[1];
        if (distance < minDis) {
            minDis = distance;
        }
        if (!visited[other - 1]) {
            dfs_minScore(minDis, visited, other, edgeMap);
        }
    }
}
```

这样`minDis`就是最后的答案了。统计：

```cpp
int minDistance = INT_MAX;
vector<bool> visited(n, false);
dfs_minScore(minDistance, visited, 1, edgeMap);
return minDistance;
```

但是，遇到了tle。之前用java的时候就没这样，但是cpp就有了。。。

所以，我尝试把dfs改成这样：

```cpp
static void dfs_minScore(int &minDis, vector<bool> &visited, int curr, map<int, vector<vector<int>>> edgeMap) {
    visited[curr - 1] = true;
    vector<vector<int>> neighbors = edgeMap[curr];
    for (auto neighbor : neighbors) {
        auto other = neighbor[0];
        if (!visited[other - 1]) {
	        continue;
        }
        auto distance = neighbor[1];
        if (distance < minDis) {
            minDis = distance;
        }
		dfs_minScore(minDis, visited, other, edgeMap);
    }
}
```

如果other已经是visited，那么就不访问了。这下，直接报错了。。。

估计是因为，即使已经访问了，也要记一下路径的长度才行，不然会错过一些case。

那咋办？我搜了一下，发现可以把map改成unordered\_map，这样会快一点。

确实快了，但是变成内存超了。所以接着搜。

首先，邻居的信息可以不用vector存，用pair存。然后，这里最大的内存占用户其实是edgeMap。所以，将传递方式改成引用，就只会分配一次了：

```cpp
static void dfs_minScore(int &minDis, vector<bool> &visited, int curr, unordered_map<int, vector<pair<int, int>>> edgeMap) {
    visited[curr - 1] = true;
    for (auto neighbor : edgeMap[curr]) {
        const auto other = neighbor.first;
        const auto distance = neighbor.second;
        if (distance < minDis) {
            minDis = distance;
        }
        if (!visited[other - 1]) {
            dfs_minScore(minDis, visited, other, edgeMap);
        }
    }
}

int Solution::minScore(int n, vector<vector<int> > &roads) {
    // build map for edges
    unordered_map<int, vector<pair<int, int>>> edgeMap;
    for (auto road: roads) {
        const int node1 = road[0];
        const int node2 = road[1];
        const int distance = road[2];
        edgeMap[node1].emplace_back(node2, distance);
        edgeMap[node2].emplace_back(node1, distance);
    }
    int minDistance = INT_MAX;
    vector<bool> visited(n, false);
    dfs_minScore(minDistance, visited, 1, edgeMap);
    return minDistance;
}
```

其实，这也给我们一个启示：优化内存的时候，如果出现递归调用，要注意一下传入的参数。参数里面通常有占用量比较大的。

- [ ] #TODO tasktodo1730049795339 另外发现一个使用并查集解决的：[2492. 两个城市间路径的最小分数 - 力扣（LeetCode）](https://leetcode.cn/problems/minimum-score-of-a-path-between-two-cities/solutions/2005130/java61hao-miao-bing-cha-ji-by-zhy-sc0rj/)，有时间看一看。 ➕ 2024-10-28 ⏫ 🆔 mx0rav 

