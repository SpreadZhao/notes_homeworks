---
num: "54"
title: "Spiral Matrix"
link: "https://leetcode.cn/problems/spiral-matrix/description/"
tags:
  - leetcode/difficulty/medium
---

# 题解

这题我是硬写的，就是走一个蛇形，然后一个个遍历。

那问题点就在于，怎么实现蛇形遍历。有几个要点要注意。

第一个问题是，每次绕圈的起点在哪儿，我画一下：

![[Projects/leetcode/resources/Drawing 2025-01-25 00.58.20.excalidraw.svg]]

我们发现，第一次绕圈的起点是0, 0，第二次起点是1, 1。不难想象到，如果这个数组再大一点，下一次的起点就是3, 3。

**因此，在第k次绕圈的时候，起点就是k, k。**

然后，是循环跳出的条件。我们能准确控制第几次绕圈能退出吗？简单想想，发现没那么容易。因为你**不确定遍历是在哪个方向退出的**。总的来说，一次绕圈分成四个方向：

1. 从左上到右上；
2. 从右上到右下；
3. 从右下到左下；
4. 从左下到左上。

上面的图里是在第3种情况退出的，而下面的例子是在第1种情况退出的：

![[Projects/leetcode/resources/Pasted image 20250125010408.png]]

所以，用另一种方式判断：假设二维数组的数字个数是$m \times n$，那么**只要我已经遍历了$m \times n$个数字，就可以退出了**。

铺垫都完了，下面我们来具体看看这四个方向是怎么遍历的。

首先，for循环是不合适的，因为for循环一般需要有一个初始值。但是这里，我们在遍历的时候会动态修改i和j的值，所以用for循环会发现，你的第一个初始化的句子没有能写的。

所以我们用while循环。

1. 首先往右走。第一次循环，k = 0，起点i和j都是k，也就是0。接下来往右走，直到最后一列，也就是`while (j < n)`。同时我们也不难发现，第二次循环里，这个方向应该少走一格，也就是n - 1，而此时k = 1。因此，可以发现左上-右上的条件就是`while (j < n - k)`；
2. 然后是往下走。这次就要动i了。和刚才一样，我们能得出这个循环的条件是`while (i < m - k)`。但是，这里需要注意一下，在刚才往右走，跳出循环后，j往右多走了一格，所以得把它拽回来；同时，此时的i, j是已经被遍历过的。所以i也要走到下一格。用上面那张图来说，从1-4遍历完之后，应该从8开始。而刚跳出第一个while循环之后，i,j指向的格子其实是4右边的一格。所以如果要回到8，应该`j--, i++`。

剩下的方向同理就行了。需要注意的点：

- 每次k++之后，来一个新的循环。此时要让i和j都是k，也就是新的起点。
- 最后一次从左下-左上时，遍历的数字比其他方向少一个。因为要去掉这一圈的起点。用上面的图来说，就是从5开始，到5结束。就这一个数字，不包括1。因为1是上一次的起点。

在这个过程中，不断往结果中put值并统计数量。如果已经到达了$m \times n$，那么就返回就行了。

代码：

```cpp
vector<int> Solution::spiralOrder(vector<vector<int>> &matrix) {
    vector<int> res;
    int i = 0, j = 0, k = 0;
    int count = 0;
    const size_t m = matrix.size(), n = matrix[0].size();
    if (matrix.empty()) {
        return res;
    }
    const size_t total = matrix.size() * matrix[0].size();
    while (true) {
        i = k, j = k;
        while (j < n - k) {
            res.push_back(matrix[i][j]);
            ++count;
            ++j;
            if (count == total) { return res; }
        }
        --j;
        ++i;
        while (i < m - k) {
            res.push_back(matrix[i][j]);
            ++count;
            ++i;
            if (count == total) { return res; }
        }
        --i;
        --j;
        while (j >= k) {
            res.push_back(matrix[i][j]);
            ++count;
            --j;
            if (count == total) { return res; }
        }
        ++j;
        --i;
        while (i > k) {
            res.push_back(matrix[i][j]);
            ++count;
            --i;
            if (count == total) { return res; }
        }
        ++k;
    }
}
```

我这个做法其实就是[官解](https://leetcode.cn/problems/spiral-matrix/solutions/275393/luo-xuan-ju-zhen-by-leetcode-solution/)里的第二种方法：

```cpp
vector<int> spiralOrder(vector<vector<int>>& matrix) {
	if (matrix.size() == 0 || matrix[0].size() == 0) {
		return {};
	}

	int rows = matrix.size(), columns = matrix[0].size();
	vector<int> order;
	int left = 0, right = columns - 1, top = 0, bottom = rows - 1;
	while (left <= right && top <= bottom) {
		for (int column = left; column <= right; column++) {
			order.push_back(matrix[top][column]);
		}
		for (int row = top + 1; row <= bottom; row++) {
			order.push_back(matrix[row][right]);
		}
		if (left < right && top < bottom) {
			for (int column = right - 1; column > left; column--) {
				order.push_back(matrix[bottom][column]);
			}
			for (int row = bottom; row > top; row--) {
				order.push_back(matrix[row][left]);
			}
		}
		left++;
		right--;
		top++;
		bottom--;
	}
	return order;
}
```