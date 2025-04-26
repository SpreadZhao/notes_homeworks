---
title:
  - apply() and let() diff
date: 2025-04-25
tags:
  - kotlin
  - "#basics"
mtrace:
  - 2025-04-25
---

# apply() and let() diff

对应todo：[[Knowledge/techniques#^950500|techniques]]。

很男泵的一点是，这个问题是我去年9.9处理的，然后到今天才终于总结。

这个本来是个内存泄漏，处理的方式也很常规，用WeakReference包一下，这也不是这次的重点。这次的重点是我看到这里面写的代码。我大概描述一下：

```kotlin
fun main() {
    var a: String? = "a"
    a.let {
        a = null
        println("a: ${a.toString()}")
        println("it: ${it.toString()}")
    }
    var b: String? = "b"
    b.apply {
        b = null
        println("b: ${b.toString()}")
        println("this: ${this.toString()}")
    }
}
```

这段代码的输出是什么？首先，a和b都被设置为空了，所以它俩一定是null。但是问题是这两个it和this会怎样。

我们会（至少我是的）下意识认为，it不是空，this是空。因为下意识认为this指代b本身而it不是。

但实际上，它俩都不是空。所以在这段代码里，let和apply的行为是一模一样的。

为啥会导致这样呢？答案是，这本身就是Kotlin的语法糖。看看let和apply的实现：

```kotlin
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return block(this)
}

@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    block()
    return this
}
```

返回值的区别我们不看，就看参数，是一个lambda。这个lambda的返回值我们也不关心。我们只关心左边参数的传递方式。

let的做法是直接作为第一个参数，而apply的做法是让它作为T的扩展。这俩有啥区别？其实，本质区别就是单参数函数和扩展函数的区别。

那是啥区别呢？仅仅是调用起来不一样，底下的实现是一模一样的。比如说：

```kotlin
fun Int.plus(other: Int): Int
```

这个函数，比如可以调用`5.plus(4)`，结果是9。那么，这实际上就是在调用`plus(5, 4)`。这知识kotlin的语法糖，**把receiver变成函数的第一个参数**而已。

那么回到一开始的问题，it和this就是两个一模一样的引用，它们的指向都不会随着a和b的指向改变而改变。只不过因为kotlin的语法糖，我们能用不同的方式获取到这个引用而已。