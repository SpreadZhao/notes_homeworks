Insertion Sort

遍历一遍数组，为每一个元素找到它的位置。从第一个元素开始，由于目前还没有排序，所以它就在那里；之后第二个元素，需要和第一个元素比大小；第三个元素需要和前两个元素比大小；第四个元素需要和前三个元素比大小……

这样看起来，其实**从第二个元素开始就可以了，因为第一个元素一定是有序的**。下面给出伪代码：

![[Lecture Notes/Algorithm/resources/Pasted image 20230531124013.png]]

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