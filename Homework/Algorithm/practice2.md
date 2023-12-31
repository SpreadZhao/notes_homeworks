# 1. 实验题目

## 1.1 Matrix-chain Product

Given the dimension of a sequence of matrices in an array `arr[]`, where the dimension of the **ith** matrix is `(arr[i-1] * arr[i])`, the task is to find the most efficient way to multiply these matrices together such that the total number of element multiplications is minimum.

## 1.2 Longest Common Subsequence

Given two strings, **S1** and **S2**, the task is to find the length of the longest subsequence present in both of the strings.

**Note:** A subsequence of a string is a sequence that is generated by deleting some characters (possibly 0) from the string without altering the order of the remaining characters. For example, “abc”, “abg”, “bdf”, “aeg”, ‘”acefg”, etc are subsequences of the string “abcdefg”.

## 1.3 Longest Common Substring

Given two strings ‘X’ and ‘Y’, find the length of the longest common substring.

## 1.4 Max Sum

You are given `n` integers (there may be negative ones but not all) $a_1,a_2,\cdots,a_n$ , determine `i` and `j` which maximize the sum from $a_i$ to $a_j$.

Example: (-2, 11, -4, 13, -5, 2), then i = 2 and j = 4, cause the sum ranged in that is 20 which is the maximum.

# 2. 实验目的

学习动态规划思想。

# 3. 实验设计与分析

## 3.1 Matrix-chain Product

本题在Geeks for Geeks上有比较详细的解释：

[Matrix Chain Multiplication | DP-8 - GeeksforGeeks](https://www.geeksforgeeks.org/matrix-chain-multiplication-dp-8/)

假设有A, B, C, D四个矩阵，我们的目的是**算它们的乘积**。然而，在线性代数里就学过，**只有满足这样关系的两个矩阵才能相乘**：

```
A is m * n, and B is n * p, then AB is m * p.
```

而题目给的是**固定顺序的矩阵的乘积计算**。因此，它们的行和列一定满足：前一个的列是后一个的行。因此，我们才能用`(arr[i-1] * arr[i])`这种方式来定义它们的属性。比如：

```c
arr[] = {40, 20, 30, 10, 30}
```

这代表了4个矩阵ABCD，它们的dimention分别是：

$$
40 \times 20,\ 20 \times 30,\ 30 \times 10,\ 10 \times 30
$$

那么问题来了。如果我们老老实实地算，也就是先算AB。这样的话，我们一共需要做$40 \times 20 \times 30$次乘法运算(可以自己画一个小矩阵证明一下)。此次运算结束之后，我们就有了一个$40 \times 30$的矩阵。然后将它和C相乘，又需要做$40 \times 30 \times 10$次运算，得到一个$40 \times 10$的矩阵。。。

然而，如果我们通过**加括号**的方式改变它们的时间顺序，能不能少算几次乘法呢？我们在线性代数学过，矩阵相乘的时间顺序可以改变，但是空间顺序不可以。比如这样：

$$
(A(BC))D
$$

这下，我们只需要执行20\*30\*10 + 40\*20\*10 + 40\*10\*30次操作就能够算完了。下面的问题就是，**我们需要找到一种加括号的方式，使得运算的次数最少**。

### 3.1.1 Recursion

说是加括号，实际是什么呢？我们看看下面这种加括号的方式：

$$
A(BC)D
$$

这和上面那个完全是一个式子。但是注意，我们其实是将这个矩阵序列**砍成了三份**。而这样的操作实际上是砍了两刀之后做到的。因此，我们讨论的所谓的**操作**应该是砍的每一刀，而不是加的每一个括号。

现在考虑第一刀。如果这个序列有N个数，那么显然我能够砍的位置有N-1个。**而砍出来的子序列，又可以递归地继续砍下去**。因此，我们可以考虑使用递归的方式来做这道题。然而，我们想要说明哪种方式是最快的，依然要**列举出所有的情况**，所以这是一个暴力递归。

现在考虑第一刀。假设我们砍在了AB之间：

$$
A|BCD
$$

我们分成了两个序列。那么总的最短次数`minCount(ABCD)`其实就是**它两个子序列的递归之和，再加上这两个序列之间相乘的消耗：**`minCount(A) + minCount(BCD) + C`。而在每一次子递归中，我们都将这个新的序列再砍一刀，继续算，直到它砍不了，也就是只剩一个元素为止。此时因为只有一个元素，计算的消耗就是0。

然而，即使是这样还不够。因为我们以上只是算了**第一刀砍在AB之间的情况**。如果要继续砍的话，那一定是一个循环，遍历了所有的位置才可以。另外在子递归中，对于每一个它的子序列，我们也要循环到每一个能砍的位置，来统计它的子序列的每一个情况。对于每一种统计好的值，我们从其中取出最小值，就是最终的答案了。

**我们让`i`是这个序列中的第一个矩阵；`j`是这个序列中最后一个矩阵**。而`k = a`表示这一刀砍在了第a个矩阵的后面，因此k迭代的范围是`i <= k < j`。下一个问题是，这个矩阵最终的消耗是多少？假设我们有这样的砍法： ^47dd0c

$$
ABCD|EFG
$$

那么最终**常量时间的**结果的运算次数一定是`A的行 * 刀两边的行列(因为它俩一定相等) * G的列`。将上面的变量带入到这个结构，我们能够得到最终的代码：

```kotlin
val count = minCount(p, i, k) + minCount(p, k + 1, j) + p[i - 1] * p[k] * p[j]
```

左边的序列就是从第i个一直到刀的左边，也就是k的值；而右边的序列就是从第k+1个开始到第j个。而运算的常量时间也是如此。在每一个子序列的子递归的for循环中，我们都找到了对于这个子序列来说，它最少的运算次数是多少，然后将它返回给上级。这样在最外层的函数中，我们就知道了这两个子序列的情况。根据以上说明，下面给出完整代码：

```kotlin
fun minCount(p: IntArray, i: Int, j: Int):Int {  
    if(i == j) return 0  
    var min = Int.MAX_VALUE  
    for(k in i until j){  
        val count = minCount(p, i, k) + minCount(p, k + 1, j) + p[i - 1] * p[k] * p[j]  
        if(count < min) min = count  
    }  
    return min  
}
```

### 3.1.2 Dynamic Programming

在上面的解决方案中，我们会发现，**很多已经算过的东西我们都要重复进行计算**。比如两个相邻的矩阵组成的序列，它们时不时就会被算一下，因为很多切法在子递归的时候都有可能会cover到这两个更子的序列：

![[Homework/Algorithm/resources/Pasted image 20230525155029.png|center|450]]

因此，我们如果能记住**第i个矩阵到第j个矩阵目前最好的情况是怎样**的话，就不用重复进行算了。这里，i和j还是[[#^47dd0c|前面]]的含义，而我们又创建了一个二维数组`dp`，它表示，从目前看来，我们已经算出的，从第i个矩阵到第j个矩阵组成的序列需要的最少的计算次数。因此，对于每次子递归，我们最终的目的就是更新`dp[i][j]`。

就加上这么一个操作，我们就完成了！只需要再注意如何初始化等一些小细节就可以了：

```kotlin
private fun dpCore(p: IntArray, i: Int, j: Int, dp: Array<IntArray>): Int {  
    if(i == j) return 0  
    if(dp[i][j] != -1) return dp[i][j]  
    dp[i][j] = Int.MAX_VALUE  
    for(k in i until j){  
        dp[i][j] = min(
			dp[i][j], 
			dpCore(p, i, k, dp) + dpCore(p, k + 1, j, dp)
			+ p[i - 1] * p[k] * p[j]
		)  
    }  
    return dp[i][j]  
}
```

### 3.1.3 Iteration Edition

既然能使用记忆的方法做，也就一定能用迭代的方法做。看看我们之前的核心语句：

```kotlin
dpCore(p, i, k, dp) + dpCore(p, k + 1, j, dp) + p[i - 1] * p[k] * p[j]
```

分为三段。前两段，其实就是**刀的左边和刀的右边的最好的结果**。其实，在上面的方法中，我们已经能保证，存下来的其实一定就已经是最好的情况了。不然我们也不会写这句话：

```kotlin
if(dp[i][j] != -1) return dp[i][j]
```

只要不是初始值，就可以使用。那就代表，**在我用这个值的时候，此时里面存的一定就已经是最好的情况**。因此，我们只要保证从最小的范围开始计算，当考虑大范围的时候，小范围的答案就一定是最优的。

![[Homework/Algorithm/resources/Pasted image 20230525164005.png|center|700]]

我们通过这个例子来解释一下。当前我们在算从A到C这个序列的最优解。显然，刀的位置可以是AB之间或者BC之间。我们能看到，如果刀放在AB之间，**最后**的花费是：

$$
[A..A] + [B..C] + const = 0 + 360 + 5 \times 10 \times 12 = 960
$$

> 其中，360来自之前小范围计算的结果。

这其实相当于这样加括号：

$$
(A)(BC)
$$

而如果我们把刀放在BC之间，最后的花费就变成了：

$$
	[A..B] + [C..C] + const = 150 + 0 + 5 \times 3 \times 12 = 330
$$

显然，这种方式比960效率更高。因此我们最终把表格中的值更新成了330。~~*当然，以后还是可能会更新的*~~。

下面介绍迭代计算时遵守的规则。由于我们要让范围从小到大，因此我们最外层的循环是在控制**矩阵序列的长度**。题的输入是n，而按照那种输入的格式，矩阵的个数就是n - 1个。因此，我们长度的迭代范围：

```kotlin
2 <= l < n
```

然后，在每次循环中，我们让i是起点，j是终点（通过计算保证i到j这个序列的长度一直是l）。然后让k依然是之前刀的位置。下面可以直接给出代码了：

```kotlin
fun minCount3(p: IntArray): Int {  
    val n = p.size  
    val dp = Array(n) { IntArray(n) { -1 } }  
    for (i in 1 until n) dp[i][i] = 0  
    for (l in 2 until n) {  
        for (i in 1 until n - l + 1) {  
            val j = i + l - 1  
            if (j == n) continue  // 这句我感觉能删掉，j永远不可能是n，不然就爆了
            dp[i][j] = Int.MAX_VALUE  
            for (k in i until j) {  
                val q = dp[i][k] + dp[k + 1][j] + p[i - 1] * p[k] * p[j]  
                if (q < dp[i][j]) dp[i][j] = q  
            }  
        }  
    }  
    return dp[1][n - 1]  
}
```

## 3.2 Longset Common Subsequence

讲解：[Longest Common Subsequence (LCS) - GeeksforGeeks](https://www.geeksforgeeks.org/longest-common-subsequence-dp-4/)，同时此题也是leetcode的1143号题：[Longest Common Subsequence - LeetCode](https://leetcode.com/problems/longest-common-subsequence/)

### 3.2.1 Bruce Force 1

首先还是暴力求解。最好的办法，就是**找到这个字符串的所有子序列**，然后一一比对即可。首先的问题是，如何找到一个字符串的所有子序列呢？我们用一种很简单的方式来想：**对于这个字符串的每个字符，我都可以选择要或者不要**。因此如果字符串的长度是n的话，它的子序列就是$2^n$个：‘

![[Homework/Algorithm/resources/Pasted image 20230414121657.png]]

> 图片来自：[(38条消息) 暴力递归——打印一个字符串的全部子序列_递归 给定一个数字的字符串,输出这个字符串的所有子串,子串的顺序和原先字符串中_爱敲代码的Harrison的博客-CSDN博客](https://blog.csdn.net/weixin_44337241/article/details/122143663)

因此，我们能大致想象出这棵递归树该如何用代码构建：

```kotlin
fun generate(i){
	generate(i) // 不要下标为i的字符
	generate(i) // 要下标为i的字符
}
```

然而，具体的实现需要关注很多细节。这里给出一种实现方式：

```kotlin
private fun generateHelper(str: String, sub: String, index: Int, subs: ArrayList<String>) {  
    if(index == str.length) {  
        subs.add(sub)  
        return  
    }  
    generateHelper(str, sub, index + 1, subs)  
    generateHelper(str, sub + str[index], index + 1, subs)  
}
```

这里的参数非常多，我们来一一说明：

* `str`：要查找的原始字符串，这个值一直不会变，比如abc；
* `sub`：当前**已经要了的所有字符组成的序列**，一开始是`""`；
* `index`：指向当前字符。这里特别需要注意，在两个子递归中写的`sub`和`sub + str[index]`分别代表我不要`str[index]`和我要`str[index]`，而接下来传入的`index + 1`代表**我在子递归中是要确定下一个字符是否被要的**。这里其实可以使用循环，而这种传入下个参数的方式将循环变为了纯递归；
* `subs`：在递归树的最深层被插入元素，表示已经确定完每一个字母是不是要了。

当`index == str.length`时，代表我已经判断完`str`中的每一个字母是否要了。因此此时可以将确定号的子序列加入到集合中。接下来，我们将这个递归函数封装一下，把初始条件设置好：

```kotlin
private fun generateSubs(str: String): List<String> {  
    val subs = ArrayList<String>()  
    generateHelper(str, "", 0, subs)  
    return subs  
}
```

最后就可以调用这个函数了。另外，我们又用了一次暴力比较来找到那个最长的公共子序列：

```kotlin
fun longestCommonSubsequence(str1: String, str2: String): String {  
    val subs1 = generateSubs(str1)  
    val subs2 = generateSubs(str2)  
    var lcs = ""  
    for(sub1 in subs1) {  
        for(sub2 in subs2) {  
            if(sub1 == sub2 && sub1.length > lcs.length) lcs = sub1  
        }  
    }  
    return lcs  
}
```

### 3.2.2 Bruce Force 2

除了上面这种纯纯暴力的递归，我们还可以想一个聪明点的。假设有两个序列X和Y，它们的长度分别是m和n：

![[Homework/Algorithm/resources/Drawing 2023-04-14 12.45.27.excalidraw.png]]

> 其中蓝色区域是**已经算出来最长公共序列的区域**，我们记目前的长度为`L(0..m-2, 0..n-2)`。

那么我们能发现有以下两种情况：

* 如果`X[m - 1] == Y[n - 1]`，那么新的最长公共子序列长度是`L(0..m-2, 0..n-2) + 1`；
* 如果`X[m - 1] != Y[n - 1]`，那么新的最长公共子序列需要进行比较：

  ```kotlin
  L(0..m-1, 0..n-1) = max(
	  L(0..m-2, 0..n-1),
	  L(0..m-1, 0..n-2)
  )
  ```

为什么需要比较呢？难道是原来的L不行吗？可别忘了，我们是在匹配序列而不是字串。虽然这两个位置的字符不相等，**但这两个字符都可能和对面字符串中之前的某些字符匹配上**。因此我们需要将情况考虑周全。当这棵树递归到最深层时，一定是m和n中的某一个变成了0，代表序列空了。此时返回0即可。L的增长其实靠的就是上面第一种情况里加的那个1。

```kotlin
private fun lcs(x: String, y: String, m: Int, n: Int): Int {  
    if(m == 0 || n == 0) return 0  
    return if(x[m - 1] == y[n - 1]) lcs(x, y, m - 1, n - 1) + 1  
    else max(lcs(x, y, m, n - 1), lcs(x, y, m - 1, n))  
}
```

> 注意这里传的参数是m-1或者n-1而不是-2，是因为m和n表示长度。序列如果是`0..m-2`，那它的长度就是m-1，所以应该传前者。

### 3.2.3 DP with Recursion

同理，由于这个`lcs()`函数会很多次重复地调用，算很多次重复的操作，比如下面的例子：

![[Homework/Algorithm/resources/Pasted image 20230414155241.png]]

在这个例子中`L(AXY, AYZ)`就被调用了两次。因此，我们可以使用动态规划来记住它们的答案。具体的实现方式和上一题一模一样，都是用`dp`数组来记住已经算过的东西：

```kotlin
private fun lcsdp(x: String, y: String, m: Int, n: Int, dp: Array<IntArray>): Int {  
    if(m == 0 || n == 0) return 0  
    if(dp[m][n] != -1) return dp[m][n]  
    dp[m][n] = if(x[m - 1] == y[n - 1])  
                    lcsdp(x, y, m - 1, n - 1, dp) + 1  
               else  
                    max(lcsdp(x, y, m, n - 1, dp), lcsdp(x, y, m - 1, n, dp))  
    return dp[m][n]  
}
```

### 3.2.4 DP

那么问题来了，我能不能不用递归，一个循环就能够实现呢？答案是可以的！其实，我们一直都只是在用这个`dp`数组里的值而已，那么我们想一想如果`X[i] == Y[j]`成立的话，代表着什么呢？那是不是就是代表着`dp[i][j] = dp[i - 1][j - 1] + 1`啊！这和我们上面讨论的情况是一模一样的；另外如果不成立，那就代表`dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])`。

为什么会这样呢？我们思考一下：**其实就算是用递归，也是要递归到最深层，再往回算**。那么我们一开始就从最深处出发，不是也可以吗！只是，我们需要注意一些细节：

* 我们会发现，`dp`中的每个元素都会向它的**上方、左上方、左边**获取数据。所以我们需要在一开始有一行和一列为铺垫； #question/coding/practice 为什么？因为**没有长度也算长度**！
* `i`和`j`此时并不是真正的数组下标，而是表示X的第i个字符和Y的第j个字符

配合上面网站的讲解，我在这里直接给出完整的代码：

```kotlin
fun longestCommonSubsequence(str1: String, str2: String): Int {  
    val m = str1.length; val n = str2.length  
    val dp = Array(m + 1){IntArray(n + 1)}  
    for(i in 0..m) {  
        for(j in 0..n) {  
            if(i == 0 || j == 0) dp[i][j] = 0  
            else if(str1[i - 1] == str2[j - 1]) dp[i][j] = dp[i - 1][j - 1] + 1  
            else dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])  
        }  
    }  
    return dp[m][n]  
}
```

## 3.3 Longest Common Substring

这道题和上道题唯一的区别是：**这道题只能向左上方要数据**！因为是字串而不是子序列，所以两个指针必须同时往左去，才代表之前的序列。而只要当前的字符不相等，那么直接可以肯定：**这两个符串从i到j的结果就是0**：

```kotlin
fun longestCommonSubstring(x: String, y: String, m: Int, n: Int): Int {  
    val dp = Array(m + 1){IntArray(n + 1)}  
    var res = 0  
    for(i in 0..m){  
        for(j in 0..n){  
            if(i == 0 || j == 0) dp[i][j] = 0  
            else if(x[i - 1] == y[j - 1]){  
                dp[i][j] = dp[i - 1][j - 1] + 1  
                res = max(res, dp[i][j])  
            }  
            else  
                dp[i][j] = 0  
        }  
    }  
    return res  
}
```

另一个区别是，我们不返回最后的`dp[m][n]`，而是返回这个数组中的最大值，为什么呢？因为前者`dp[m][n]`一定同时也是最大值！如果统计的是子序列的话，对于整个两个字符串来说，最长子序列的长度本身也一定是最长的；而最长字串却没有这样的特点。

## 3.4 Max Sum

此题为leetcode第53题：[Maximum Subarray - LeetCode](https://leetcode.com/problems/maximum-subarray/)

此题是一道比较小的动态规划，并且也不需要大量的空间。我们从头开始遍历，只要满足：

```kotlin
pre + curr > curr
```

这是什么意思？也就是这个序列前面已经算出来的最优解如果加上当前的值比当前这个数大。我们可能会有这样的疑问：如果想让这个和越来越大，不应该是新的结果比原来的大吗？也就是：

```kotlin
pre + curr > pre
```

注意，我们的目的并不是从头开始寻找某个序列，而是**不一定从哪个位置开始**。因此前者的条件只要不成立，那么curr的位置就应该是一个新的候选人的开始。为什么这么说？如果不成立的话，那就代表`pre <= 0`，这意味着之前的序列对于当前的curr是一个**拖后腿（可以想想`[8, -5, -5, 2]`这个序列）**的存在。所以我们只能往后进行统计。并且更重要的是，我们每一次保存的并不是全局的最优解，而是**对于当前这个元素的局部最优解**。如果条件不成立的话，那么这个最优解将成为新的开始。

于此同时，我们在每一次确定了新的局部最优解后，都要不断选择最大的，从而最终确定全局的最优解。

```kotlin
fun maxSum(arr: IntArray): Int {  
	var b = arr[0]  // 局部最优解
	var sum = b  // 全局最优解
	for (i in 1 until arr.size) {  
		b = if(b + arr[i] > arr[i]) b + arr[i] else arr[i]  
		if(b > sum) sum = b  
	}  
	return sum  
}
```

另外，本题的代码可以简化为：

```kotlin
fun maxSum(arr: IntArray): Int {  
	var b = arr[0]  
	var sum = b  
	for (i in 1 until arr.size) {  
		if(b > 0) b += arr[i]  
		else b = arr[i]  
		if(b > sum) sum = b  
	}  
	return sum  
}
```

# 4. 实验环境

* OS: Windows 11
* IDE: IDEA
* Language: Kotlin

# 5. 项目测试

![[Homework/Algorithm/resources/Pasted image 20230425124105.png]]

![[Homework/Algorithm/resources/Pasted image 20230425124137.png]]

![[Homework/Algorithm/resources/Pasted image 20230425124206.png]]

![[Homework/Algorithm/resources/Pasted image 20230425124523.png]]