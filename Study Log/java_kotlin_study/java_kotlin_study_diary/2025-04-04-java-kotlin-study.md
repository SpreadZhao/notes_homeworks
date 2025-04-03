---
title: kotlin-init-issue
date: 2025-04-04
tags: 
mtrace: 
  - 2025-04-04
---

# kotlin-init-issue

和Kotlin类的初始化相关：

```kotlin
class InitTest {

    init {
        test()
    }

    private fun test() {
        println("a: $a")
    }

    private var a = 1

}

fun main() {
    val test = InitTest()
}
```

下面这段代码，输出是多少？如果是1就没这篇文章了。答案是0。这个问题是我在做一个需求的时候偶然发现的，做一个实施特征的项目，在init块执行的时候要初始化一些东西，然后要用到这个类的一个成员。当时debug了半天，发现它是null。

怎么解决呢？把a的初始化挪到init块上面就行了。注意，是init块上面，挪到test函数和init块中间可不行，函数的定义和运行时可不是一个东西：

```kotlin
class InitTest {

    private var a = 1

    init {
        test()
    }

    private fun test() {
        println("a: $a")
    }

}

fun main() {
    val test = InitTest()
}
```

- [ ] #TODO tasktodo1743697425200 Kotlin构造方法执行的顺序？ ➕ 2025-04-04 🔼 🆔 sag04b 