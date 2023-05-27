# 1. 实验题目

## 1.1 Knapsack Problem.

There are 5 items that have a value and weight list below, the knapsack can contain at most 100 Lbs. Solve the problem using back-tracking algorithm and try to draw the tree generated.

![[Homework/Algorithm/resources/Pasted image 20230430092113.png]]

## 1.2 8-Queen Problem

Solve the 8-Queen problem using back-tracking algorithm.

# 2. 实验目的

回溯法

# 3. 实验设计与分析

## 3.1 Knapsack Problem

回溯法其实就是暴力的计算。只不过我们可以尽量使用一些技巧来减少回溯的次数。KMP算法就是这样。

## 3.2 8-Queen Problem

首先，要介绍一下8皇后问题。在一块$8 \times 8$的棋盘上，摆放8个皇后。要求**任意两个皇后不能处于同一行；不能处于同一列；不能处于同一斜线**。显然，使用回溯法就可以对这个问题进行求解。

首要的问题是，我们如何表示8个皇后的位置。使用二维数组？可以，但没必要。因为两个皇后的行和列必须都不一样，所以我们可以**只用一个一维数组，让下标表示行，里面存的值表示列**。这样，每个元素的下标必然不一样，那么就一定能够保证所有的皇后都不会处于同一行。接下来，我们需要给出判断两个皇后是否处于同一列或者同一斜线的方法：

```kotlin
private fun isSameColumn(i1: Int, i2: Int) = arr[i1] == arr[i2]  
private fun isSameBias(i1: Int, i2: Int) = abs(i1 - i2) == abs(arr[i1] - arr[i2])
```

显然，元素的值相等代表列相同；下标的距离和值的距离相等代表处于同一斜线。有了这两个方法，我们才能继续我们的推理。

每当我摆放了一个皇后，都要看看这个皇后是否和其他的皇后冲突。首先我们从第一个皇后开始。它有8种选择。这显然就是一个for循环：

```kotlin
for (i in arr.indices) {  
    arr[n] = i   // n = 1 
}
```

上面的代码代表，我为第一个皇后考虑了从0号到7号共8个**列**。当摆放好第一个皇后之后，就应该考虑第二个皇后放在哪里。因此，应该将n换成n + 1重新走一遍这个逻辑。这显然是一个递归：

```kotlin
private fun putQueen(n: Int) {  
	for (i in arr.indices) {  
		arr[n] = i  
		putQueen(n + 1)  
	}  
}
```

我们来走一遍。首先，是第一个皇后（n = 0），按照代码逻辑，他会被放到(0, 0)：

![[Homework/Algorithm/resources/Drawing 2023-05-26 23.42.09.excalidraw.png|center|300]]

之后会走到递归中，然后在n = 1的时候，依然会让`arr[1] = 0`：

![[Homework/Algorithm/resources/Drawing 2023-05-26 23.48.54.excalidraw.png|center|300]]

那问题来了：这个皇后不应该被放在这里。因此，**我们需要为每个皇后考虑到它能够放的列**。那么根据什么考虑呢？自然是**已经放好的皇后**。因此，我们需要有一个检测的方法：

```kotlin
private fun putQueen(n: Int) {   
	for (i in arr.indices) {  
		arr[n] = i  
		if (noCollision(n)) putQueen(n + 1)  
	}  
}

private fun noCollision(n: Int): Boolean {  
    for (i in 0 until n) {  
        if (isSameColumn(i, n) || isSameBias(i, n)) return false  
    }  
    return true  
}
```

有了这个方法，我们再来走一遍：对于n = 1的这个第二个皇帝，它依然会放在第0列。**然而，当放完之后，`noCollision()`检测出了问题，因此，for循环需要继续进行，为它考虑其他位置**。另外，即使通过了`noCollision()`的检测，我们也不应该break掉这个for循环。拿第一个（n = 0）皇后举例子，**难道你把她放在(0, 0)没出问题，就不考虑\[1 .. 7\]这些位置了**？

![[Homework/Algorithm/resources/Drawing 2023-05-27 00.01.05.excalidraw.png|center|300]]

上图是本程序最一开始的几步。然而即使是第一个符合条件的解，第二行的皇后也不是在第三列，而是在第五列：

![[Homework/Algorithm/resources/Pasted image 20230527000410.png|center|300]]

下面是本程序的全部代码。其中包含一些用于观察实时输出的逻辑。注意，当摆放到最后一个皇后时，我们需要输出一下本次的结果：

```kotlin
class Queen8(private var count: Int = 0) {  
    companion object {  
        private const val MAX = 8  
    }  
    private val arr = IntArray(MAX) { Int.MIN_VALUE }  
    private val mockArr = Array(MAX) { IntArray(MAX) { 0 } }  
    
    fun run(): Int {  // Entry of the whole program
        putQueen(0)  // Start from the first queen
        return count  
    }
      
    private fun putQueen(n: Int) {  
        if (n == MAX) {  // Last Queen
            count++  
            mock()  
            return  
        }  
        for (i in arr.indices) {  
            arr[n] = i  
//            mock()  
            if (noCollision(n)) putQueen(n + 1)  
        }  
    }  
    
    private fun noCollision(n: Int): Boolean {  
        for (i in 0 until n) {  
            if (isSameColumn(i, n) || isSameBias(i, n)) return false  
        }  
        return true  
    }  
    
    private fun isSameColumn(i1: Int, i2: Int) = arr[i1] == arr[i2]  
    
    private fun isSameBias(i1: Int, i2: Int) = abs(i1 - i2) == abs(arr[i1] - arr[i2])  
  
    private fun mock() {  
        for (i in arr.indices) {  
            if (arr[i] != Int.MIN_VALUE) mockArr[i][arr[i]] = 1  
        }  
        mockArr.forEach {  
            it.forEach { num ->  
                print("$num ")  
            }  
            println()  
        }  
        println("---------------------")  
        mockArr.forEach {  
            it.fill(0)  
        }  
    }  
}
```

# 4. 实验环境

* OS: Windows 11
* IDE: IDEA
* Language: Kotlin

# 5. 项目测试

## 5.1 Knapsack Problem.

![[Homework/Algorithm/resources/Pasted image 20230511234033.png]]

![[Homework/Algorithm/resources/Pasted image 20230511234647.png]]

## 5.2 A simple scheduling problem

![[Homework/Algorithm/resources/Pasted image 20230511235001.png]]

## 1.3 Single-source / All-pairs shortest paths

```kotlin
fun shortest() {  
	val I = Int.MAX_VALUE  
	val edges = arrayOf(  
		intArrayOf(0, -1, 3, I, I),  
		intArrayOf(I, 0, 3, 2, 2),  
		intArrayOf(I, I, 0, I, I),  
		intArrayOf(I, 1, 5, 0, I),  
		intArrayOf(I, I, I, -3, 0)  
	)  
	println("before:")  
	edges.forEach {  
		it.forEach { num ->  
			if (num != I) print("$num\t")  
			else print("I\t")  
		}  
		println()  
	}  
	println("bellman-ford:")  
	ShortestPath().shortestPath(edges).forEach { num -> print("$num ") }  
	println()  
	ShortestPath().floyd(link = edges)  
	println("floyd:")  
	edges.forEach {  
		it.forEach { num ->  
			if (num != I) print("$num\t")  
			else print("I\t")  
		}  
		println()  
	}  
}
```

![[Homework/Algorithm/resources/Pasted image 20230511235307.png|center|200]]