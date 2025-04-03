---
title:
  - Lambda leak mem
date: 2025-04-03
tags: 
mtrace:
  - 2025-04-03
---

# Lambda leak mem

做搜索同构需求时，偶然发现了一个之前修复的内存泄漏。本身没什么技术含量，还是context被单例持有了。例子是这样的：

```kotlin
fun preloadCards(context: Context) {
	IdleTaskDispatchers.getDefault().addIdleTask(object : NameRunnable("task") {
		override fun run() {
			doCreateHolder(context)
		}
	})
}
```

预加载Holder，这个很好理解。创建Holder的时候需要Context，因为需要用Inflater加载。因此，这里直接把context传到了这个匿名类中（其实这不算lambda，但也差不多）。

这里泄漏的点也很好理解：IdleTaskDispatchers是一个单例类，持有Context那必须被泄漏。

解决办法也很简单，把context用弱引用包一层就行了：

```kotlin
fun preloadCards(context: Context) {
	val contextRef = WeakReference(context)
	IdleTaskDispatchers.getDefault().addIdleTask(object : NameRunnable("task") {
		override fun run() {
			doCreateHolder(contextRef.get() ?: return)
		}
	})
}
```

todo:

- [ ] #TODO tasktodo1743695568197 为什么用弱引用能修复这个问题？➕ 2025-04-03 ⏫ 🆔 xu5yzo 
- [ ] #TODO tasktodo1743695584721 为什么Context生命周期短？泄漏了有什么问题？ ➕ 2025-04-03 🆔 0kpo8j ⏫ 
- [ ] #TODO tasktodo1743695750938 补充一下强软弱虚[[Study Log/java_kotlin_study/java_kotlin_study_diary/reference#Java的引用类型|reference]] ➕ 2025-04-03 🔼 🆔 g1s0uy 