---
title:
  - LiveData get the recent available data immediately on observation
date: 2025-02-20
tags:
  - livedata
  - bugfix
mtrace:
  - 2025-02-20
---

# LiveData get the recent available data immediately on observation

这个也是老生长谈的LiveData数据倒灌问题：[LiveData 数据倒灌问题：我们需要知道的那些事儿_livedata数据倒灌怎么解决-CSDN博客](https://blog.csdn.net/qq_42751010/article/details/141645254)。

我在业务开发中遇到的情况是这样的：在外层Feed流中，先点作者头像，会展开个人页浮层。此时会上报`enter_pgc`埋点。然后关掉这个浮层，进入合集内流，然后再点这个头像，会发现，这次点开之后会**同时上报2个`enter_pgc`埋点！这显然是不符合预期的；甚至，其中一个埋点还缺少了logpb**。

- [x] #TODO tasktodo1739984342379 补一下图片。 🆔 k0zx4c 🔺 ➕ 2025-02-20 ✅ 2025-04-25

> 图片就不补了，说一下我是怎么发现的，首先，两次埋点通过byteio就能看到。而上报这个埋点的位置是确定的，那debug一下就行了。两次断在这里的时候，对应View的hash是不一样的。所以一下就意识到，是因为之前在外流先点了个人页，启动了一个View导致的。

首先，外流的个人页和内流的个人页是同一个实现的不同实例，反正是两个一样类型的View。而进入内流之后，外流的那个个人页虽然被关掉了，但是是没有被回收的。

在初始化View的时候，会先注册LiveData的观察，然后发起查询个人页数据的请求。在请求成功之后，会修改LiveData，表明成功进入。在成功进入的回调中会上报`enter_pgc`埋点。发起请求用的是ViewModel，**这是被两个实例所共用的**。

因此，在这个bad case中，时序如下：

1. 外层View展开，注册了一个observer，然后发起请求，返回成功，修改LiveData。此时状态为Success；
2. 外层View上报`enter_pgc`埋点，此次上报符合预期。
3. 之后进入内流，内层View展开，再次注册了一个observer。此时，因为LiveData的特性，注册的时候会立刻收到一个Success状态并执行回调。也会上报`enter_pgc`。但是，这个时候埋点参数还没有绑定上（绑定的时机要晚于注册），所以此次上报丢失了logpb信息；另外，这次上报本身是不应该存在的，因为网络请求还没发出去，凭什么执行success回调？
4. 然后，内层View也发起网络请求，再次收到Success并执行回调，最后又上报了一次`enter_pgc`。此时埋点信息已经绑定上，所以此次上报符合预期。

总结，这三次上报中，中间的那次是不应该存在的。根本原因就是LiveData的数据倒灌：

```kotlin
viewModel.userCount.observe(viewLifecycleOwner) {
	Log.i("Study-LiveData", "[o1] user count: $it")
}
getUserBtn.setOnClickListener {
	viewModel.addUser()
}
addObserverBtn.setOnClickListener {
	if (!observerAdded) {
		observerAdded = true
		viewModel.userCount.observe(viewLifecycleOwner) {
			Log.i("Study-LiveData", "[o2] user count: $it")
		}
	}
}
```

我们用代码来模拟一下。o1是一开始注册好的observer，对应外流的View。o2是内流的View注册的observer，在点击按钮之后才会添加。我们首先正常不断点击getUserBtn，日志是这样的：

![[Study Log/android_study/android_diary/resources/Pasted image 20250220005448.png]]

然后我们点击一下addObserverBtn。会发生什么呢？

![[Study Log/android_study/android_diary/resources/Pasted image 20250220005529.png]]

我们发现，observer刚注册上，就立刻收到了一个最近的更新6。其实，这个逻辑在LiveData的observe方法的注释里都已经写了：

> When data changes while the owner is not active, it will not receive any updates. <u>If it becomes active again, it will receive the last available data automatically.</u>

解决方法其实已经有很多了，参考文章里的自定义Event就是一种。但是，因为修改整个加载个人页的那个成本有点高（其实是我要下班了，快点修完吧。。。），所以我最后用的方式是，每次observe之前，先remove掉其它observer。具体实现就不说了，一行代码。