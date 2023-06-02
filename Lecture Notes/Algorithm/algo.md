Insertion Sort

遍历一遍数组，为每一个元素找到它的位置。从第一个元素开始，由于目前还没有排序，所以它就在那里；之后第二个元素，需要和第一个元素比大小；第三个元素需要和前两个元素比大小；第四个元素需要和前三个元素比大小……

这样看起来，其实**从第二个元素开始就可以了，因为第一个元素一定是有序的**。下面给出伪代码：

![[Lecture Notes/Algorithm/resources/Pasted image 20230531124013.png|500]]

kotlin代码：

```kotlin
class InsertionSort {  
	fun sort(array: IntArray) {  
		for (i in 1 until array.size) {  
			val key = array[i]; var j = i - 1  
			while (j >= 0 && array[j] > key) {  
				array[j + 1] = array[j]  
				j--  
			}  
			array[j + 1] = key  
		}  
	}  
}
```

loop-invariant

使用循环不变式证明算法的正确，需要三个步骤：

* Initialization: 循环开始前就正确
* Maintenance: 在某一次循环之前是正确的
* Termination: 循环结束时的结果时正确的

![[Lecture Notes/Algorithm/resources/Pasted image 20230531124245.png|500]]

使用loop-invariant证明插入排序的正确性：

* 在循环开始之前，由于从第二个元素（j = 2）开始迭代，所以初始序列就是A\[1\]，它包含了原始序列的元素A\[1\]，并且**有序（这就代表最终的输出在一开始是对的）**；
* 当一次循环结束时，我们给A\[j\]找到了它的位置。这证明我们中间的结果A\[1..j\]也是有序的；
* 当j = n + 1时，我们跳出了循环，此时A\[1..j-1\]就是我们要的输出，所以结果也是有序的。

Divide and Conquer

![[Lecture Notes/Algorithm/resources/Pasted image 20230531130007.png|500]]

> **recursivevly千万不能落**！

Merge Sort

[[Lecture Notes/Algorithm/ea#3.2 Sorting(Merge sort)]]

```kotlin
class MergeSort {  
	fun sort(array: IntArray) {  
		core(array, 0, array.lastIndex)  
	}  
	  
	private fun core(array: IntArray, low: Int, high: Int) {  
		if (low < high) {  
			val mid = (low + high) / 2  
			core(array, low, mid)  
			core(array, mid + 1, high)  
			merge(array, low, mid, high)  
		}  
	}  
	  
	private fun merge(array: IntArray, low: Int, mid: Int, high: Int) {  
		val left = array.copyOfRange(low, mid + 1) + intArrayOf(Int.MAX_VALUE)  
		val right = array.copyOfRange(mid + 1, high + 1) + intArrayOf(Int.MAX_VALUE)  
		var i = 0; var j = 0  
		for (k in low .. high) {  
			if (left[i] < right[j]) {  
				array[k] = left[i]  
				i++  
			} else {  
				array[k] = right[j]  
				j++  
			}  
		}  
	}  
}
```

Using substitution method to solve the recurrence:

$$
T(n) = 4T(\frac{n}{2}) + 100n
$$

* Guess: we guess that: $T(n) \leqslant cn^3$

For $k = \dfrac{n}{2}$, this inequality should also be correct, which means:

$$
T(k) \leqslant ck^3 \Longrightarrow T(\frac{n}{2}) \leqslant c \cdot (\frac{n}{2})^3
$$

Put this in to the origin equation:

$$
\begin{array}{rcl}
T(n) & = & 4T(\dfrac{n}{2}) + 100n \\
& \leqslant & 4c \cdot (\dfrac{n}{2})^3 + 100n \\
& = & (\dfrac{c}{2})n^3 + 100n \\
& = & cn^3 - [(\dfrac{c}{2})n^3 - 100n] \\
& \leqslant & cn^3
\end{array}
$$

**一道容易错的题**：

![[Lecture Notes/Algorithm/resources/Pasted image 20230531215550.png|500]]

Recurtion Tree

![[Lecture Notes/Algorithm/resources/Pasted image 20230531220321.png|500]]

![[Lecture Notes/Algorithm/resources/Pasted image 20230531220806.png|500]]

![[Lecture Notes/Algorithm/resources/Pasted image 20230531220936.png|500]]

> **对于递归树的高度，它的底数一定是大于一的**！

[[Lecture Notes/Algorithm/ea#^6cdbcb|Master Theorem]]

[[Homework/Algorithm/practice2#3.4 Max Sum|Maximum Subarray Problem]]，使用Divide and Conquer:

```kotlin
class MaximumSubarray {  
	fun maxSumDivideConquer(array: IntArray): Int {  
		return coreDC(array, 0, array.lastIndex)  
	}  
	  
	private fun coreDC(array: IntArray, low: Int, high: Int): Int {  
		return if (low == high) array[low]  
		else {  
			val mid = (low + high) / 2  
			val leftMaxSum = coreDC(array, low, mid)  
			val rightMaxSum = coreDC(array, mid + 1, high)  
			val crossMaxSum = coreDCCrossing(array, low, mid, high)  
			maxOf(leftMaxSum, rightMaxSum, crossMaxSum)  
		}  
	}  
	  
	private fun coreDCCrossing(array: IntArray, low: Int, mid: Int, high: Int): Int {  
		var leftMaxSum = Int.MIN_VALUE  
		var sum = 0  
		for (i in mid downTo low) {  
			sum += array[i]  
			if (sum > leftMaxSum) leftMaxSum = sum  
		}  
		var rightMaxSum = Int.MIN_VALUE  
		sum = 0  
		for (j in mid + 1 .. high) {  
			sum += array[j]  
			if (sum > rightMaxSum) rightMaxSum = sum  
		}  
		return leftMaxSum + rightMaxSum  
	}  
}
```

核心思想就是，对于一个序列，它的最大和只能出现在以下三种情况：

* 只包含在前一半
* 只包含在后一半
* 跨越前一半和后一半

所以，我们要分别计算这三种情况，最后三个数比大小。第一种，如果只包含前一半，那么直接使用原函数递归就可以，也就是从`low`到`mid`，对于后一半也是一样，从`mid + 1`到`high`。而比较麻烦的，是第三种跨越的情况。我们可以思考一下，如果是第三种，最后的答案**一定是包括`mid`和`mid + 1`这两个元素**。因此，我们使用两个循环，分别计算出包含`mid`的左边的最大和，以及包含`mid + 1`的右边的最大和：

```kotlin
private fun coreDCCrossing(array: IntArray, low: Int, mid: Int, high: Int): Int {  
	var leftMaxSum = Int.MIN_VALUE  
	var sum = 0  
	for (i in mid downTo low) {  // 左边最大和
		sum += array[i]  
		if (sum > leftMaxSum) leftMaxSum = sum  
	}  
	var rightMaxSum = Int.MIN_VALUE  
	sum = 0  
	for (j in mid + 1 .. high) {  // 右边最大和
		sum += array[j]  
		if (sum > rightMaxSum) rightMaxSum = sum  
	}  
	return leftMaxSum + rightMaxSum  
}  
```

最后返回的结果，**一定是把他们两个加起来，一定是**！！！因为如果你不加的话，等于承认结果中不包含`mid`或者不包含`mid + 1`，而这样的话其实和那三种情况的前两种是重的，我们的计算也就没有意义了。

[[Lecture Notes/Algorithm/ea#3.1 Matrix Multiplication|Matrix Multiplication]]

Heap:

[src/algo/MaxHeap.kt · SpreadZhao/leetcode - 码云 - 开源中国 (gitee.com)](https://gitee.com/spreadzhao/leetcode/blob/master/src/algo/MaxHeap.kt)

构造函数中加了一个`Int.MIN_VALUE`是因为为了空出第一个节点，这样真正的堆中的元素是从第二个元素开始算。对于**自底向上**，也就是上面代码中的构建大顶堆的方式，时间复杂度是$O(n)$，并不是$O(nlogn)$。

Priority Queue:

[src/algo/PriorityQueue.kt · SpreadZhao/leetcode - 码云 - 开源中国 (gitee.com)](https://gitee.com/spreadzhao/leetcode/blob/master/src/algo/PriorityQueue.kt)

QuickSort:

[[Homework/Algorithm/practice1#3.3 Quick Sort|practice1]]

[src/algo/QuickSort.kt · SpreadZhao/leetcode - 码云 - 开源中国 (gitee.com)](https://gitee.com/spreadzhao/leetcode/blob/master/src/algo/QuickSort.kt)

里面介绍了另一种方法来进行partition操作：

```kotlin
private fun partition2(arr: IntArray, low: Int, high: Int): Int {  
	val pivot = arr[high]  
	var i = low - 1  
	for (j in low until high) {  
		if (arr[j] < pivot) {  
			i++  
			swap(arr, i, j)  
		}  
	}  
	swap(arr, i + 1, high)  
	return i + 1  
}
```

在这个方法中，我们只需要**找出所有比`arr[high]`小的元素，并把它放在这些元素的右边即可**。走一走循环我们就能发现，我们并不关心比pivot大的元素在哪里，因为只需要有最后一句swap，就能保证所有比pivot大的元素都出现在pivot的右边。而for循环中做的就是找到所有比pivot小的元素，**并让i记住他们中的最后一个**。

Counting Sort

Counting Sort的核心思想就是：如果有5个数（包含我自己）小于等于我，那么我就可以被放在第5号。因为，从第6号开始，都一定是大于我的数字，所以我放在第五位一定是正确的。

```kotlin
class CountingSort {  
	fun sort(arr: IntArray, range: IntRange): IntArray {  
		val res = IntArray(arr.size)  
		val location = IntArray(range.last + 1) { 0 }  
		for (j in arr.indices) location[arr[j]]++  
		for (i in 2 .. range.last) location[i] += location[i - 1]  
		for (j in arr.lastIndex downTo 0) {  
			res[location[arr[j]] - 1] = arr[j]  
			location[arr[j]]--  
		}  
		return res  
	}  
}
```

第一个for循环：

```kotlin
for (j in arr.indices) location[arr[j]]++
```

是为了记录每个元素出现的次数：

![[Lecture Notes/Algorithm/resources/Pasted image 20230601182115.png|500]]

比如此时，1022就分别是1234在A中出现的次数。但是，即使这样还不行，因为我要知道**有多少个数字是小于等于我的**。而我目前只知道有多少个数字（包括自己）是等于我的。因此，下一个for循环就是为了计算这个：

```kotlin
for (i in 2 .. range.last) location[i] += location[i - 1]  
```

从第一个往后叠加，像多米诺骨牌一样，把前面的和不断累加到后面，最终C里存的就应该是我们要的，小于等于（包括自己）的数字有多少个。

![[Lecture Notes/Algorithm/resources/Pasted image 20230601182438.png|500]]

最后，我们从A的最后一个数字开始，找到它的位置。比如3，我知道整个数组中，**小于等于它（也包括它自己这个3）的数字有3个，所以它至少也要放在第三名**。因此，我们将3放在`小于等于这个3的数字的个数`名：

```kotlin
res[location[arr[j]] - 1] = arr[j]
```

> 这里需要注意，第三名是从1开始的，因此对应的下标还需要-1。

![[Lecture Notes/Algorithm/resources/Pasted image 20230601182714.png|500]]

那么，如果之后又遇到3该咋办？那么顺理成章，它应该放在第二名。所以我们直接把C中的值减掉，这样之后就能顺利放到对的位置：

```kotlin
location[arr[j]]--
```

![[Lecture Notes/Algorithm/resources/Pasted image 20230601182824.png|500]]

![[Lecture Notes/Algorithm/resources/Pasted image 20230601222257.png]]

[十种基本的排序算法 - 掘金 (juejin.cn)](https://juejin.cn/post/6844904023821123592)

[[Happy-SE-in-XDU-master/Algorithm/supplement_all#^13e1e0|supplement_all]]

Dynamic Programming

![[Lecture Notes/Algorithm/resources/Pasted image 20230602134555.png|500]]
