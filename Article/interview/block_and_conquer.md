遇到什么问题？都是怎么解决的？这篇文章总结的全部都是这样的问题。其他文件中的问题会贴出 #block_and_conquer 的标签。

```dataview
list
where contains(file.tags, "block_and_conquer")
```

# 1 测试的时候无法定位问题

在蔚来实习的时候，初期跟着测试在车上体验我们业务的全流程。那个时候经常是我手上拿着手机，然后一旦出现了问题或者触发了某个逻辑，就立刻记录一下当前的时间，等测试结束后再发给相应的开发看那个时间的日志。但是久而久之我发现，如果仅仅是记录时和分的化，依然很难定位具体的时间点。于是我想开发了一个悬浮窗，能够实时更新当前的时间，直接精确到毫秒级。然后拿着另一台手机对着这台手机排视频，这样不管什么时候出现了问题，都能在100ms的误差内报告某个逻辑的时间点。

#TODO 画饼

- [ ] WindowManager和更新UI

在开发悬浮窗的时候，我还顺便了解了[[WindowManager]]这个东西以及[[切换到主线程更新UI的方法]]。

#TODO ANR

- [x] ANR问题怎么解决的？

之后，我在自己的手机上也重新写了一个版本。但是让我意外的是，居然遇到了ANR问题。

![[Article/interview/resources/2c51e9e770fa49a34fa371f74c083cf.png]]

![[Article/interview/resources/ANR问题.png]]

这中间，我排查了很多原因。比如悬浮窗构建时Context到底要选什么？一开始我选的是Activity，后来又换成Service和Application；之后又在[这篇文章](https://blog.csdn.net/weixin_38322371/article/details/119185227)中发现，把悬浮窗的实例放在Service里，是需要在Service里用Handler来更新悬浮窗的UI的，然而我自己当时的设计是在悬浮窗内部更新UI，所以我也换成了这样的模式。之后，在公司的手机上测试，又发现了绘制的Frame的事件没有交付完毕的情况。所以我又思考下面的代码：

```kotlin
private val refreshTimeTask = object : Runnable {  
	override fun run() {  
		window.refreshTime()  
		mHandler.postDelayed(this, 1000)  
	}  
}
```

如果就这样放着的话，这个程序放在后台一段时间，悬浮窗被回收了，而这个绘制过程恰好正在进行，那不是就不会提交了吗！于是，我又搞了一个[[监听屏幕开关的Listener]]，当屏幕关闭时，就上一个锁，不更新UI，然后等屏幕亮了就把它打开并让Handler提交一次任务；除了这些，还考虑了是否应该用前台Service。。。

以上，**没有任何作用**！最终，是我把应用的省电策略调成不限制，就不会有ANR问题了。。。。。。。。。。。