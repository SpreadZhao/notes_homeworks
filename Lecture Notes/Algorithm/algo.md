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

Maximum Subarray Problem，使用Divide and Conquer:

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

