拥有rating标签的是重要的面试问题。

```dataview
list
where contains(file.tags, "question/interview")
```

# View

## onMeasure方法一般执行几次，什么情况下会执行多次

#rating/medium 

onMeasure: [[Study Log/android_study/view_create_flow#3.2 Measure|view_create_flow]]

[Android中View的绘制过程 onMeasure方法简述 附有自定义View例子 - 圣骑士wind - 博客园 (cnblogs.com)](https://www.cnblogs.com/mengdd/p/3332882.html)

![[Article/interview/resources/Pasted image 20230713103325.png]]

[(45条消息) View的三次measure,两次layout和一次draw_程序员历小冰的博客-CSDN博客](https://blog.csdn.net/u012422440/article/details/52972825)

#question/coding/practice Activity创建过程，onMeasure会执行几次？

[(45条消息) 进入Activity时，为何页面布局内View#onMeasure会被调用两次？_android onmeasure调用两次_tinyvampirepudge的博客-CSDN博客](https://blog.csdn.net/qq_26287435/article/details/123274342)

这两次，第一次是measureHierarchy触发的，第二次是performMeasure触发的：

![[Article/interview/resources/Pasted image 20230714151942.png]] ![[Article/interview/resources/Pasted image 20230714152029.png]]

经过我自己的探索，又发现了一个会重新执行onMeasure的时机。看看下面的例子：

![[Article/interview/resources/Pasted image 20230714140843.png]]

上面是一个输入框和按钮，下面是我自定义的跑马灯：

```xml
<LinearLayout  
    android:layout_width="match_parent"  
    android:layout_height="wrap_content"  
    android:orientation="horizontal">  
  
    <EditText        
	    android:id="@+id/control_marquee"  
        android:layout_width="0dp"  
        android:layout_height="wrap_content"  
        android:layout_weight="1"  
        />  
  
    <Button        
	    android:id="@+id/submit"  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:text="Submit"  
        />  
</LinearLayout>  
  
<com.example.customviewtest.MarqueeText  
    android:id="@+id/marquee"  
    android:layout_width="100dp"  
    android:layout_height="wrap_content"  
    />
```

我们输入一个可以让跑马灯跑起来的字符串：

![[Article/interview/resources/Pasted image 20230714141052.png]]

可以看到，跑马灯成功地跑了起来。但是，如果我们在EditText里插入一个换行符，也就是让这个空间的大小发生改变，此时如果我们在MarqueeText的onMeasure方法中打了断点，就能发现，居然真的执行到了这里。

**然而，如果我们将这两个控件调换一下位置**：

![[Article/interview/resources/Pasted image 20230714141545.png]]

这时，如果我们插入一个换行符，就不会执行到onMeasure方法了。

![[Article/interview/resources/Pasted image 20230714141711.png]]

因此，我们可以得到一个结论：如果布局的改变会影响当前布局的形态，位置的话，也是会重新执行onMeasure方法的。

## 点击icon到展示第一帧页面经过了哪些流程，尤其是AndroidManagerService起了什么作用

#rating/high 

应用启动流程：[[Study Log/android_study/app_boot_process|app_boot_process]]

[Android启动过程分析(图+文)-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1356506)

## View的绘制流程分为几步，从哪儿开始？哪个过程结束之后能够看到View？

重点就是这张图：[[Study Log/android_study/resources/Pasted image 20230710142551.png]]

从ViewRoot的performTraversals开始，经过measure，layout,draw 三个流程。draw流程结束以后就可以在屏幕上看到view了。

## View的测量宽高和实际宽高有区别吗？

基本上百分之99的情况下都是可以认为没有区别的。有两种情况，有区别。第一种 就是有的时候会因为某些原因 view会[[#onMeasure方法一般执行几次，什么情况下会执行多次|测量多次]]，那第一次测量的宽高 肯定和最后实际的宽高 是不一定相等的，但是在这种情况下最后一次测量的宽高和实际宽高是一致的。此外，实际宽高是在layout流程里确定的，我们可以在layout流程里将实际宽高写死 写成硬编码，这样测量的宽高和实际宽高就肯定不一样了，虽然这么做没有意义 而且也不好。

## View的measureSpec由谁决定？顶级View呢？

我们在构造View的时候，知道要传入一个LayoutParams参数。而这个参数实际上就是在XML中声明View的时候写的那些属性。这些属性在入之后，在onMeasure阶段时，就会起到约束的作用。还记得我们重写onMeasure的逻辑吗？在[[Study Log/android_study/custom_view|自定义View]]的文章中：

```java
int widthSize = MeasureSpec.getSize(widthMeasureSpec); 
int heightSize = MeasureSpec.getSize(heightMeasureSpec);
```

这里实际上就是从LayoutParams里得到的结果，要么是match_parent，要么是确切的大小数值。而这也是为什么我们要单独处理wrap_content的原因了。

另外，我们也不能毫无限制地传入LayoutParams，因为父容器一定是有一个限制的。因此，父容器的大小也是决定Spec的一个因素， #question/coding/practice  而这也是可能需要多次测量的原因。

> #question/coding/practice 看这张图：
> 
> ![[Article/interview/resources/Pasted image 20230714104239.png]]
> 
> 这里说这两个Spec是父View施加上去的。然而经过实践可知，如果我们给这个View传入一个wrap_content属性，这里得到的值就会是AT_MOST。那为啥还是父亲施加上去的，难道不是我们自己传入的requirement吗？

对于顶级的View，则稍微特殊一点。代码在ViewRootImpl中：

```java
/**
 * Figures out the measure spec for the root view in a window based on it's
 * layout params.
 *
 * @param windowSize The available width or height of the window.
 * @param measurement The layout width or height requested in the layout params.
 * @param privateFlags The private flags in the layout params of the window.
 * @return The measure spec to use to measure the root view.
 */
private static int getRootMeasureSpec(int windowSize, int measurement, int privateFlags) {
	int measureSpec;
	final int rootDimension = (privateFlags & PRIVATE_FLAG_LAYOUT_SIZE_EXTENDED_BY_CUTOUT) != 0
			? MATCH_PARENT : measurement;
	switch (rootDimension) {
		case ViewGroup.LayoutParams.MATCH_PARENT:
			// Window can't resize. Force root view to be windowSize.
			measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
			break;
		case ViewGroup.LayoutParams.WRAP_CONTENT:
			// Window can resize. Set max size for root view.
			measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
			break;
		default:
			// Window wants to be an exact size. Force root view to be that size.
			measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
			break;
	}
	return measureSpec;
}
```

## 普通View的measure过程和它的父View有关吗？如果有关，这个父View扮演了什么角色？

父View会是一个ViewGroup，因为只有ViewGroup才配拥有子节点。测量ViewGroup的过程参考[[Study Log/android_study/view_create_flow#3.2.4 ViewGroup的测量|这里]]。下面是其中WithMargins的源码：

```java
/**
 * Ask one of the children of this view to measure itself, taking into
 * account both the MeasureSpec requirements for this view and its padding
 * and margins. The child must have MarginLayoutParams The heavy lifting is
 * done in getChildMeasureSpec.
 *
 * @param child The child to measure
 * @param parentWidthMeasureSpec The width requirements for this view
 * @param widthUsed Extra space that has been used up by the parent
 *        horizontally (possibly by other children of the parent)
 * @param parentHeightMeasureSpec The height requirements for this view
 * @param heightUsed Extra space that has been used up by the parent
 *        vertically (possibly by other children of the parent)
 */
protected void measureChildWithMargins(View child,
		int parentWidthMeasureSpec, int widthUsed,
		int parentHeightMeasureSpec, int heightUsed) {
	// 第一步 先取得子view的 layoutParams 参数值
	final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

	// 然后开始计算子view的spec的值，注意这里看到 
	// 计算的时候除了要用子view的 layoutparams参数以外
	// 还用到了父view 也就是viewgroup自己的spec的值
	final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
			mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
					+ widthUsed, lp.width);
	final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
			mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
					+ heightUsed, lp.height);

	child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}


/**
 * Does the hard part of measureChildren: figuring out the MeasureSpec to
 * pass to a particular child. This method figures out the right MeasureSpec
 * for one dimension (height or width) of one child view.
 *
 * The goal is to combine information from our MeasureSpec with the
 * LayoutParams of the child to get the best possible results. For example,
 * if the this view knows its size (because its MeasureSpec has a mode of
 * EXACTLY), and the child has indicated in its LayoutParams that it wants
 * to be the same size as the parent, the parent should ask the child to
 * layout given an exact size.
 *
 * @param spec The requirements for this view
 * @param padding The padding of this view for the current dimension and
 *        margins, if applicable
 * @param childDimension How big the child wants to be in the current
 *        dimension
 * @return a MeasureSpec integer for the child
 */
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
	int specMode = MeasureSpec.getMode(spec);
	int specSize = MeasureSpec.getSize(spec);

	int size = Math.max(0, specSize - padding);

	int resultSize = 0;
	int resultMode = 0;

	switch (specMode) {
		// Parent has imposed an exact size on us
		case MeasureSpec.EXACTLY:
			if (childDimension >= 0) {
				resultSize = childDimension;
				resultMode = MeasureSpec.EXACTLY;
			} else if (childDimension == LayoutParams.MATCH_PARENT) {
				// Child wants to be our size. So be it.
				resultSize = size;
				resultMode = MeasureSpec.EXACTLY;
			} else if (childDimension == LayoutParams.WRAP_CONTENT) {
				// Child wants to determine its own size. It can't be
				// bigger than us.
				resultSize = size;
				resultMode = MeasureSpec.AT_MOST;
			}
			break;
	
		// Parent has imposed a maximum size on us
		case MeasureSpec.AT_MOST:
			if (childDimension >= 0) {
				// Child wants a specific size... so be it
				resultSize = childDimension;
				resultMode = MeasureSpec.EXACTLY;
			} else if (childDimension == LayoutParams.MATCH_PARENT) {
				// Child wants to be our size, but our size is not fixed.
				// Constrain child to not be bigger than us.
				resultSize = size;
				resultMode = MeasureSpec.AT_MOST;
			} else if (childDimension == LayoutParams.WRAP_CONTENT) {
				// Child wants to determine its own size. It can't be
				// bigger than us.
				resultSize = size;
				resultMode = MeasureSpec.AT_MOST;
			}
			break;
	
		// Parent asked to see how big we want to be
		case MeasureSpec.UNSPECIFIED:
			if (childDimension >= 0) {
				// Child wants a specific size... let them have it
				resultSize = childDimension;
				resultMode = MeasureSpec.EXACTLY;
			} else if (childDimension == LayoutParams.MATCH_PARENT) {
				// Child wants to be our size... find out how big it should
				// be
				resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
				resultMode = MeasureSpec.UNSPECIFIED;
			} else if (childDimension == LayoutParams.WRAP_CONTENT) {
				// Child wants to determine its own size.... find out how
				// big it should be
				resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
				resultMode = MeasureSpec.UNSPECIFIED;
			}
			break;
	}
	//noinspection ResourceType
	return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
```

## View的measure和onMeasure有什么关系

[[Study Log/android_study/view_create_flow#^8894fd|view_create_flow]]

## View的Measure流程

[[Study Log/android_study/view_create_flow#3.2 Measure|view_create_flow]]

![[Study Log/android_study/resources/Pasted image 20230710145603.png]]

## 自定义View中如果onMeasure方法没有对wrap_content做处理，会发生什么？怎么解决？

见[[#View的measureSpec由谁决定？顶级View呢？]]，所以实际上是和match_parent效果相同。所以，我们需要给一个默认的宽或者高。而这个值是多少？其实我们是可以动态测量出来的。[[Study Log/android_study/custom_view#4.4 Example|这就是一个例子]]。

## ViewGroup有onMeasure方法吗，为什么？

首先，没有搜索到，所以一定没有。另外，[[Study Log/android_study/view_create_flow#3.2.4 ViewGroup的测量|ViewGroup的测量流程]]中，也只是调用了子View的测量方法，所以这个方法是交给子类自己实现的。不同的viewgroup子类布局都不一样，那onMeasure索性就全部交给他们自己实现好了。

## 为什么在Activity的生命周期里无法获得测量宽高？有什么方法可以解决这个问题吗？

[Activity中获取某个View宽高信息的四种方法 - 掘金 (juejin.cn)](https://juejin.cn/post/7056659774028382245)

因为measure的过程和activity的生命周期  没有任何关系。你无法确定在哪个生命周期执行完毕以后 view的measure过程一定走完。可以尝试如下几种方法 获取view的测量宽高。

- [*] onWindowFocusChanged

该方法的含义是：View已经初始化完毕了，宽/高已经准备好了，所以此时去获取宽/高是没有问题的。

注意：onWindowFocusChanged会被调用多次，当activity的窗口得到焦点和失去焦点时均会被调用一次，具体来说，当activity继续执行（onResume）和暂停执行（onPause）时，onWindowFocusChanged均会被调用。

```kotlin
// testView可以是Activity的成员
override fun onWindowFocusChanged(hasFocus: Boolean) {  
    super.onWindowFocusChanged(hasFocus)  
    if (hasFocus) {  
        val width = testView.measuredWidth  
        val height = testView.measuredHeight  
        Log.v("MainActivity", "width: $width")  
        Log.v("MainActivity", "height: $height")  
    }  
}
```

- [*] view.post

通过post可以将一个runnable投递到消息队列的尾部，然后等待Looper调用此runnable时，View也已经初始化好了。

```kotlin
override fun onStart() {  
    super.onStart()  
    testView.post {   
		val width = testView.measuredWidth  
        val height = testView.measuredHeight  
    }  
}
```

- [*] ViewTreeObserver

使用ViewTreeObserver的众多回调也可以完成这个功能，比如使用OnGlobalLayoutListener这个接口。当View树的状态发生改变或者View树内部的View的可见性发生改变时，onGlobalLayout方法将被调用。

注意：伴随着View树的状态改变等，onGlobalLayout会被调用多次。因此需要在适当时机将监听回调移除。

```kotlin
override fun onStart() {  
	super.onStart()  
	val observer = testView.viewTreeObserver  
	observer.addOnGlobalLayoutListener(object : OnGlobalLayoutListener {  
		override fun onGlobalLayout() {  
			observer.removeOnGlobalLayoutListener(this)  
			val width = testView.measuredWidth  
			val height = testView.measuredHeight  
		}  
	})  
}
```

```ad-error
title: 警告

这种方式会导致程序崩溃，目前在调查原因。
```

## layout和onLayout方法有什么区别

[[Study Log/android_study/view_create_flow#3.3 Layout|view_create_flow#layout]]

layout是确定本身view的位置而onLayout是确定所有子元素的位置。layout里面就是通过[[Study Log/android_study/view_create_flow#^42577a|setFrame]]方法设设定本身view的四个顶点的位置。这4个位置以确定 自己view的位置就固定了。

然后就调用onLayout来确定子元素的位置。view和viewgroup的onlayout方法都没有写。都留给我们自己给子元素布局。

## draw方法有几个步骤

比较重要的步骤有4个：[[Study Log/android_study/view_create_flow#^2bf32e|view_create_flow]]。然而，其中的第二步和第五步通常情况下是可以跳过的。为什么跳过可以看看源码。

#TODO 分析跳过的源码

## setWillNotDraw方法有什么用

这是View的方法：

```java
/**
 * If this view doesn't do any drawing on its own, set this flag to
 * allow further optimizations. By default, this flag is not set on
 * View, but could be set on some View subclasses such as ViewGroup.
 *
 * Typically, if you override {@link #onDraw(android.graphics.Canvas)}
 * you should clear this flag.
 *
 * @param willNotDraw whether or not this View draw on its own
 */
public void setWillNotDraw(boolean willNotDraw) {
	setFlags(willNotDraw ? WILL_NOT_DRAW : 0, DRAW_MASK);
}
```

用于设置标志位的 也就是说 如果你的自定义view不需要draw的话，就可以设置这个方法为true。这样系统知道你这个view 不需要draw 可以**优化执行速度**。viewgroup一般都默认设置这个为true，因为viewgroup多数都是只负责布局。

不负责draw的。而view 这个标志位 默认一般都是关闭的。

## 自定义View有什么需要注意的？

1. wrap_content
2. padding

下面是一个自定义的圆形：

```kotlin
class CircleView : View {  
    private val mColor = Color.RED  
    private val mPaint = Paint(Paint.ANTI_ALIAS_FLAG).apply { color = mColor }  
  
    constructor(context: Context) : super(context)  
    constructor(context: Context, attrs: AttributeSet) : super(context, attrs)  
    constructor(context: Context?, attrs: AttributeSet?, defStyleAttr: Int) : super(  
        context,  
        attrs,  
        defStyleAttr  
    )  
  
    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {  
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)  
  
        val widthMode = MeasureSpec.getMode(widthMeasureSpec)  
        val heightMode = MeasureSpec.getMode(heightMeasureSpec)  
  
        val widthSize = MeasureSpec.getSize(widthMeasureSpec)  
        val heightSize = MeasureSpec.getSize(heightMeasureSpec)  
  
        if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {  
            setMeasuredDimension(200, 200);  
        } else if (widthMode == MeasureSpec.AT_MOST) {  
            setMeasuredDimension(200, heightSize)  
        } else if (heightMode == MeasureSpec.AT_MOST) {  
            setMeasuredDimension(widthSize, 200)  
        }  
    }  
  
    override fun onDraw(canvas: Canvas?) {  
        super.onDraw(canvas)  

		// padding在xml中设置后，就会成为getPaddingXXX的返回值
        val w = width - paddingLeft - paddingRight  
        val h = height - paddingTop - paddingBottom  
        val r = min(w, h) / 2  
        canvas?.drawCircle(  
            (paddingLeft + w / 2).toFloat(),  
            (paddingTop + h / 2).toFloat(),  
            r.toFloat(),  
            mPaint  
        )  
    }  
}
```

# Touch Event

## 触摸事件传递的流程都有哪些可能？

首先，触摸事件是一个**一次性**的东西，也就是在传递的过程中，如果被某个View给拦截并消费了，就会终止。而这个消费的过程就写在onTouchEvent()中，所以这整个过程看起来就像一条L型的链子：

![[Article/interview/resources/Pasted image 20230725094136.png|500]]

而如果任何一个View都没有消费这个事件，那么这个事件就会最终重新回到Activity，变成一个U型的链：

![[Article/interview/resources/Pasted image 20230725094305.png|500]]

## 这些MotionEvent在触摸的时候都可能会触发多少次？

我们可以在dispatchEvent和onTouchEvent上都打上日志来看一看：

```
dispatch, ev: MotionEvent { action=ACTION_DOWN, ...
onTouch, ev: MotionEvent { action=ACTION_DOWN, ...
dispatch, ev: MotionEvent { action=ACTION_MOVE, ...
onTouch, ev: MotionEvent { action=ACTION_MOVE, ...
dispatch, ev: MotionEvent { action=ACTION_MOVE, ...
onTouch, ev: MotionEvent { action=ACTION_MOVE, ...
... ...
dispatch, ev: MotionEvent { action=ACTION_UP, ...
onTouch, ev: MotionEvent { action=ACTION_UP, ...
```

可以看到，DOWN只有一次，而中间可能会经过若干次MOVE，最后来一个UP。这也是非常符合我们的常识的。

![[Article/interview/resources/Pasted image 20230725100859.png|500]]

## 如果要实现拖拽，需要如何重写方法？

最简单地，只需要重写onTouchEvent()方法就可以了。当按下时，记住点击的位置，然后MOVE时，不断算出新的位置并更新：

```kotlin
override fun onTouchEvent(event: MotionEvent): Boolean {  
	when (event.action) {  
		MotionEvent.ACTION_MOVE -> {  
			val x = event.rawX.toInt()  
			val y = event.rawY.toInt()  
			val destX = x - mLastFocusX  
			val destY = y - mLastFocusY  
			windowParams.x += destX  
			windowParams.y += destY  
			windowManager.updateViewLayout(this, windowParams)  
			mLastFocusX = x  
			mLastFocusY = y  
		}  
		MotionEvent.ACTION_DOWN -> {  
			mLastFocusX = event.rawX.toInt()  
			mLastFocusY = event.rawY.toInt()  
			performClick()  
		}  
	}  
	return true  
}
```

上面的例子是一个可以拖拽的悬浮窗。当move时，首先算出新的位置保存到windowParams中，然后用它来更新悬浮窗。而如果我们想要设计一个可以全局拖拽的普通View的话，可以使用layout()方法：

```kotlin
override fun onTouchEvent(event: MotionEvent): Boolean {  
    when (event.action) {  
        MotionEvent.ACTION_MOVE -> {  
            val x = event.rawX.toInt()  
            val y = event.rawY.toInt()  
            val destX = x - mLastFocusX  
            val destY = y - mLastFocusY  
            val left = left + destX  
            val top = top + destY  
            val right = right + destY  
            val bottom = bottom + destY  
            layout(left, top, right, bottom)  
            mLastFocusX = x  
            mLastFocusY = y  
        }  
        MotionEvent.ACTION_DOWN -> {  
            mLastFocusX = event.rawX.toInt()  
            mLastFocusY = event.rawY.toInt()  
            performClick()  
        }  
    }  
    return true  
}
```

另外，我们还可以在onDraw()方法里去处理，并在更新完之后调用invalidate()来触发绘画。可以参考自定义气泡的例子：[[Article/story/2023-07-25#自定义QQ气泡View|2023-07-25]]。

```ad-caution
我们要注意一下getX()和getRawX()的区别。前者获取的是相对于**屏幕**左上角的横坐标，而后者是相对于收到这个事件的**View**左上角的横坐标。
```

## 什么时候才需要重写dispatchTouchEvent()方法？

`dispatchTouchEvent`是一个用于分发触摸事件的方法，它负责将触摸事件传递给目标View和ViewGroup的方法。大部分情况下，你不需要重写`dispatchTouchEvent`方法，因为它在`View`和`ViewGroup`类中已经有了默认的实现，会自动处理触摸事件的分发。

但有时候，你可能需要重写`dispatchTouchEvent`方法来实现一些特定的触摸事件分发逻辑。以下是一些需要重写`dispatchTouchEvent`方法的情况：

1.  事件拦截：如果你希望在触摸事件到达目标View之前拦截并处理触摸事件，可以重写`dispatchTouchEvent`方法，然后在适当的时候返回`true`来拦截事件，或者返回`false`将事件传递给目标View。
2.  自定义事件分发逻辑：在某些特殊情况下，你可能希望根据自己的业务需求定制触摸事件的分发逻辑。例如，你可能需要根据一些条件来决定是否将触摸事件传递给子View，或者在某些情况下将触摸事件传递给父View。
3.  事件日志记录：有时候你可能需要在应用程序中记录触摸事件的信息，以便进行调试或分析。通过重写`dispatchTouchEvent`方法，你可以在事件分发过程中添加日志记录。

值得注意的是，如果你决定重写`dispatchTouchEvent`方法，通常你需要在方法中实现一些复杂的逻辑来正确地处理触摸事件的分发。因此，在没有特定需求的情况下，最好不要随意重写`dispatchTouchEvent`方法，以免引入不必要的复杂性和潜在的错误。大多数情况下，可以通过在`onTouchEvent`方法中处理触摸事件来满足常规需求。

## 什么时候才需要重写onInterceptTouchEvent()方法？

`onInterceptTouchEvent`方法用于拦截子View的触摸事件，它是在`ViewGroup`类中定义的。当一个`ViewGroup`包含多个子View时，如果你希望在某些特定情况下拦截子View的触摸事件，并阻止其传递给子View的`onTouchEvent`方法，就可以重写`onInterceptTouchEvent`方法。

一般情况下，你不需要经常重写`onInterceptTouchEvent`方法，因为它的默认实现已经满足大多数情况。但以下情况可能需要重写该方法：

1.  滑动冲突解决：当`ViewGroup`中包含多个可滑动的子View（例如`ScrollView`、`RecyclerView`等）时，可能会出现滑动冲突。如果你希望在某些情况下阻止子View的滑动事件，就可以在`onInterceptTouchEvent`方法中实现相关逻辑来解决滑动冲突。
2.  手势处理：在某些场景下，你可能需要在`ViewGroup`中检测特定的手势，例如双击、长按等，来触发一些自定义操作。在这种情况下，你可以重写`onInterceptTouchEvent`方法来实现手势的拦截和处理。
3.  自定义触摸事件分发逻辑：如果你的`ViewGroup`需要定制触摸事件的分发逻辑，例如根据某些条件来决定是否拦截触摸事件，就可以在`onInterceptTouchEvent`方法中实现相应的逻辑。

需要注意的是，当你重写`onInterceptTouchEvent`方法时，需要谨慎处理触摸事件的拦截逻辑，以免引入不必要的滑动冲突或触摸事件的混乱。正确处理触摸事件的拦截逻辑能够提供更好的用户体验，并确保各个子View能够正确响应用户的触摸操作。

最后，与`dispatchTouchEvent`方法一样，在没有特定需求的情况下，最好不要随意重写`onInterceptTouchEvent`方法，以免引入复杂性和潜在的错误。在大多数情况下，通过在`onTouchEvent`方法中处理触摸事件已经能够满足常规需求。

# Looper

## 一个线程有几个Looper？