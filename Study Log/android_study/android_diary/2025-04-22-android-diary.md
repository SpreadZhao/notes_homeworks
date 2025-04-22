---
title: View的属性拿不到？
date: 2025-04-22
tags: 
mtrace: 
  - 2025-04-22
---

# View的属性拿不到？

当View的属性获取不到时，要立刻反应出来是没测量完。拿RecyclerView的bind举例子。如果我们要在RecyclerView bind的时候，获取holder的itemView的属性，就需要post一下。因为bind流程就在layout流程里。

这里再注意一下，我们是怎么改View的宽高的？是通过设置LayoutParams。但是，LayoutParams的设置，你自己点进去看就知道，还是进行一次requestLayout。像width, height，measuredWidth等属性，都是要下一帧才能拿到的。所以，你懂了吧！

另外，这里介绍一下measuredWidth和width的区别。measuredWidth获取的是在调用了View的measure()之后得到的结果。注意，仅仅是measure。所以，这个东西最多的使用情况是自定义父View的时候，在onMeasure里面需要测量子View，所以就调用一下子View的measure（这个你看一下FrameLayout，里面就有这种逻辑）。这样不管你是马上要在onMeasure用，还是之后在onLayout里用，就都能拿到了。

width的作用就是给使用者的。获取的是view已经显示在屏幕上之后的结果。在onLayout的时候，view的宽高还可能发生改变。比如View因为某些规则被拉长，挤扁。这就和measure的时候确定的要求不太一样了。

这也能验证，measure确定的只是每个View的“需求”。而真正决定最终属性的，是在layout。

参考：[What is the difference between getWidth/Height() and getMeasuredWidth/Height() in Android SDK? - Stack Overflow](https://stackoverflow.com/questions/8657540/what-is-the-difference-between-getwidth-height-and-getmeasuredwidth-height-i)

还有，post为什么能保证在下一帧执行？看看post的源码吧。

View.post(Runnable)执行时，会根据View当前状态执行不同的逻辑：当View还没有执行测量、布局、绘制时，View.post()会将Runnable任务放入一个任务队列中以待后续执行；反之，当View已经执行了测量、绘制后，Runnable任务会直接通过AttachInfo中的Handler执行（UI线程中的Handler）。总之View.post()能够保证提交的任务是在View测量、绘制之后执行，所以可以得到正确的宽高。

当前View只有在依附到View树之后，调用View.post()中的任务才有机会执行；反之只是new一个View实例，并未关联到View树的话，那么该View.post()中的Runnable任务永远都不会得到执行。

这里的代码分析随便搜一下就有了。