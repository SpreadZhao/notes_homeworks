# 以前的总结

![[Article/interview/android_interview#1. Activity|android_interview]]

# 生命周期Practice

实操：在一个Activity上打开另一个Activity，执行的流程：

![[Study Log/android_study/resources/Pasted image 20230717110057.png]]

我们发现，当A执行了onPause之后，并没有立刻执行onStop，而是在第二个Activity执行完onCreate -> onStart -> onResume之后才会执行onStop。

将SecondActivity换成Dialog的形式之后：

![[Study Log/android_study/resources/Pasted image 20230717110634.png]]

会发现MainActivity的onStop不会执行，因为此时用户是能看见这个Activity的。

当在Dialog显示的时候，点击空白处以关闭Dialog，回到MainActivity时：

![[Study Log/android_study/resources/Pasted image 20230717111508.png]]

我们也能发现，当MainActivity真的已经显示在最顶层（onResume）之后，Dialog才会进行销毁，也就是onStop和onDestroy。

现在把Dialog再换成普通的Activity，退出时的操作：

![[Study Log/android_study/resources/Pasted image 20230717111803.png]]

通过以上的情况，我们能总结出来：**当Activity要发生切换时，一个Activity的onPause方法就是为另一个Activity让步的**。在一个Activity的onPause执行完毕后，另一个Activity会**立刻**试图执行到onResume以显示。当显示完毕后，之前让步的Activity才会继续往下走流程。

# 启动模式Practice

在starndard模式下，连续启动了三次我自己：

![[Study Log/android_study/resources/Pasted image 20230717114110.png]]

每次的ID都不一样，所以每次都会创建出一个新的Activity到返回栈中，将原来的压下去。

在singletop模式下，无论我启动多少次我自己，都只有最一开始创建的信息：

![[Study Log/android_study/resources/Pasted image 20230717134047.png]]

然而，如果我在MainActivity和SecondActivity之间反复横跳（**不是通过返回键**）的话，结果又不一样了：

![[Study Log/android_study/resources/Pasted image 20230717134943.png]]

现在MainActivity和SecondActivity都是singletop模式，然而我们发现依然会创建新的实例。也就是这个模式下不在栈顶的Activity还是会创建新的实例的。

现在把这两个Activity都换成singletask模式：

![[Study Log/android_study/resources/Pasted image 20230717140410.png]]

MainActivity在反复横跳的过程中，只会创建一次了。然而SecondActivity却会创建多次。这是因为，我们在SecondActivity中启动MainActivity，系统检测到MainActivity是singletask的，并且**它此刻就在栈下面**。所以直接就调用类似返回的逻辑了：

![[Study Log/android_study/resources/Pasted image 20230717140703.png]]

于是再启动SecondActivity的时候，就会走创建一个Activity的流程了。