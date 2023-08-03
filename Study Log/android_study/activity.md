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

在starndard模式下，连续启动了三次我自己： ^83ed41

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

# 启动模式介绍

[Android 面试黑洞——当我按下 Home 键再切回来，会发生什么？_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1CA41177Se/?spm_id_from=333.999.0.0&vd_source=64798edb37a6df5a2f8713039c334afb)

当我们点击了手机上的那个方块键，或者手机上滑之后，看到的这一个个的，是什么呢？

![[Study Log/android_study/resources/msedge_9YVkfrEkA0.gif|500]]

答案是**Task**。当我们点击了一个桌面上的App图标时，那个配置了MAIN+LAUNCHER的Activity就会被启动：

![[Study Log/android_study/resources/Pasted image 20230803143730.png]]

同时，这个Activity也会被放进系统新创建出的一个Task里：

![[Study Log/android_study/resources/Pasted image 20230803143807.png|300]]

比如，下图中展示的，就是后台的四个Task。其中最下面的是用户正在打开的前台Task：

![[Study Log/android_study/resources/Pasted image 20230803143932.png|500]]

**每一个Task都有一个返回栈来管理这些Activity**，当我们在一个任务中不停点返回键，这些Activity就会依次被关闭（onDestroy），直到最后一个Activity被关闭，这个Task的生命周期也就结束了。然而，即使这个Task不存在了，我们在切到最近任务时，依然可以看见它：

![[Study Log/android_study/resources/Pasted image 20230803144331.png|500]]

这并不代表这个程序没有被杀死，而是只是系统为这个应用保留了一个“残影”。当我们点击它时，**加载的动画是程序启动的动画，而不是从后台跳出来的动画**：

![[Study Log/android_study/resources/msedge_KC4qAJBV6z.gif|500]]

接下来，我们来说一下跨进程，跨应用启动的过程。我们新建两个应用，ActivityTest1和ActivityTest2。ActivityTest1里面有一个启动ActivityTest2的MainActivity的按钮：

```kotlin
class MainActivity : ComponentActivity() {  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        setContent {  
            ActivityTest1Theme {  
                // A surface container using the 'background' color from the theme  
                Surface(  
                    modifier = Modifier.fillMaxSize(),  
                    color = MaterialTheme.colorScheme.background  
                ) {  
//                    Greeting("Android")  
                    LaunchModeTest()  
                }  
            }        
		}    
	}  
}  
  
@Composable  
fun LaunchModeTest() {  
    val context = LocalContext.current  
  
    Column {  
        Button(onClick = {  
            context.startActivity(  
                Intent().setComponent(  
                    ComponentName("com.example.activitytest2", "com.example.activitytest2.MainActivity")  
                )  
            )  
        }) {  
            Text(text = "Start Other App's Activity")  
        }  
    }
}
```

而ActivityTest2里面只有一个TextField用来输入文字：

```kotlin
class MainActivity : ComponentActivity() {  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        setContent {  
            ActivityTest2Theme {  
                // A surface container using the 'background' color from the theme  
                Surface(  
                    modifier = Modifier.fillMaxSize(),  
                    color = MaterialTheme.colorScheme.background  
                ) {  
//                    Greeting("Android")  
                    Edit()  
                }  
            }        
		}    
	}  
}  
  
@OptIn(ExperimentalMaterial3Api::class)  
@Composable  
fun Edit() {  
    var text by remember {  
        mutableStateOf("")  
    }  
    Column {  
        TextField(  
            value = text,  
            onValueChange = { text = it }  
        )  
    }  
}
```

我们首先启动ActivityTest2，在里面输入一串文字，然后通过ActivityTest1里的按钮来启动这个ActivityTest2。现在来看看效果：

![[Study Log/android_study/resources/scrcpy_EuKV6nkAF6.gif|300]]

可以看到，我们自己启动的ActivityTest2的MainActivty，和通过ActivityTest1的按钮启动的ActivityTest2的MainActivity，它们的数据是**不共享的**！这和我们之前的Practice中的内容也是一致的：[[#^83ed41]]

接下来，用一个动画来演示一下这个跨应用的情况：

![[Study Log/android_study/resources/msedge_1GBwQHaXpH.gif|500]]

就像视频中说的，*为什么这么设计*？为什么别的应用的Activity，可以被我这个应用任意支配呢？甚至不会影响那个提供Activity应用本身？我们现在考虑一种使用情况：我在QQ中点击了一个邮箱链接，想发送邮件。那么此时的操作显然是从QQ当前的Activity，跳转到了邮箱App中的Activity。就像这样：

![[Study Log/android_study/resources/scrcpy_Oa3SK6GZYs.gif|300]]

然而，**如果我不想这样操作了呢**？或者说，我不想发邮件了呢？从用户的角度来想，**按一下返回不就好了嘛！并且，在绝大多数情况下，我也希望按下返回之后，我回到的应用应该是QQ而不是Outlook**。我们来实验一下：

![[Study Log/android_study/resources/scrcpy_0N2PbOQVeu.gif|300]]

果然回到了原来的应用！而这也就是安卓默认启动模式standard的特点：Activity在start的时候都会创建出一个新的实例。而这样的特性，使得它在给其它应用提供功能时变得更加灵活，且不会影响自己；另外，我们也能注意到，**这个写邮件的Activity和QQ是相关的，因为它就是从QQ打开的；和Outlook本身却是不相关的，因为我只是想写个邮件，并没有用到其它Outlook中的功能**。你可能会问：如果我手滑点了一下返回，那我写的邮件不就没了？别担心，Outlook早就考虑了这一点。我们回到QQ之后，再打开Outlook，是可以看到它为我们保存了一份草稿的：

![[Study Log/android_study/resources/Pasted image 20230803160857.png|300]]

这个功能的实现就很多样了，可能是定时备份，也可能是在Activity退出的时候执行。

> #TODO 
> 
> - [ ] Activity退出的时候，哪一个阶段适合做这样的操作？

```ad-info
我没有用视频中短信和通讯录的例子，因为我的手机里短信和通讯录是合在一起的一个应用；相反，我的邮件倒是和他Standard的例子是一样的（视频中邮件被用作SingleTask的例子）。
```

接下来，我们再来看一看SingleTask的例子。还是之前的AT1和AT2。我们仅仅是将AT2的MainActivity的启动模式换一下：

```xml
<activity  
    android:name=".MainActivity"  
    android:exported="true"  
    android:label="@string/app_name"  
    android:theme="@style/Theme.ActivityTest2"  
    android:launchMode="singleTask"  
    >  
    <intent-filter>        
	    <action android:name="android.intent.action.MAIN" />  
        <category android:name="android.intent.category.LAUNCHER" />  
    </intent-filter>
</activity>
```

换成了SingleTask之后，重新运行一下AT2，然后输入一串字符，之后从AT1的按钮里启动AT2：

![[Study Log/android_study/resources/scrcpy_a70JXZkBr7.gif|300]]

这下结果就完全不一样了！不是一个新的Activity，而是原来带有我们输入的字符的Activity。我们再深入了解一下：修改AT2的代码，加入一个新的Activity：

```kotlin
class EditActivity : ComponentActivity() {  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        setContent {  
            ActivityTest2Theme {  
                // A surface container using the 'background' color from the theme  
                Surface(  
                    modifier = Modifier.fillMaxSize(),  
                    color = MaterialTheme.colorScheme.background  
                ) {  
                    Edit()  
                }  
            }        
		}    
	}  
}  
  
@OptIn(ExperimentalMaterial3Api::class)  
@Composable  
fun Edit() {  
    var text by remember {  
        mutableStateOf("")  
    }  
    Column {  
        TextField(  
            value = text,  
            onValueChange = { text = it }  
        )  
    }  
}
```

我们将输入框的部分移到了一个新的EditActivity中，并让MainActivity能够启动它：

```kotlin
class MainActivity : ComponentActivity() {  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        setContent {  
            ActivityTest2Theme {  
                // A surface container using the 'background' color from the theme  
                Surface(  
                    modifier = Modifier.fillMaxSize(),  
                    color = MaterialTheme.colorScheme.background  
                ) {  
                    StartEdit()  
                }  
            }        
		}    
	}  
}  
  
@Composable  
fun StartEdit() {  
    val context = LocalContext.current  
    Column {  
        Button(onClick = {  
            val intent = Intent(context, EditActivity::class.java)  
            context.startActivity(intent)  
        }) {  
            Text(text = "StartEdit")  
        }  
    }
}
```

然后，我们把MainActivity的启动模式改回Standard，把EditActivity的启动模式改成SingleTask：

```xml
<activity  
    android:name=".EditActivity"  
    android:exported="false"  
    android:label="@string/title_activity_edit"  
    android:theme="@style/Theme.ActivityTest2"  
    android:launchMode="singleTask"  
/>  
<activity  
    android:name=".MainActivity"  
    android:exported="true"  
    android:label="@string/app_name"  
    android:theme="@style/Theme.ActivityTest2">  
    <intent-filter>        
	    <action android:name="android.intent.action.MAIN" />  
        <category android:name="android.intent.category.LAUNCHER" />  
    </intent-filter>
</activity>
```

AT1的代码不用修改，还是启动MainActivity就好。我们来观察一下实际的情况。**首先是，确保清除掉AT2的后台，然后启动AT1**：

![[Study Log/android_study/resources/scrcpy_DSSfpDEr50.gif|300]]

一切正常。按照之前我们介绍的逻辑，应该是这样的：

![[Study Log/android_study/resources/Drawing 2023-08-03 16.56.52.excalidraw.png]]

**但是，如果我们启动了AT2，再进行一遍流程的话：**

![[Study Log/android_study/resources/scrcpy_YcrPiQW6tG.gif|300]]

*为什么中间出现了两个AT2的MainActivity*？如果我们深入了解了SingleTask的机制，就能够知道：**之前的那张图其实是错误的**！AT2的EditActivity是一个SingleTask，所以它的启动可不是简简单单地向当前的任务中堆一个Activity而已。

我们先把这个问题放一放。~~因为，我们是用ComponentName的方式启动它的~~[^错误的原因]。我们先做一个比较简单的案例。修改AT2的代码，再增加一个SingleTask的Activity： ^9b0410

```kotlin
class EditActivity2 : ComponentActivity() {  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        setContent {  
            ActivityTest2Theme {  
                // A surface container using the 'background' color from the theme  
                Surface(  
                    modifier = Modifier.fillMaxSize(),  
                    color = MaterialTheme.colorScheme.background  
                ) {  
                    Text(text = "I'm EditActivty2")  
                }  
            }        
		}    
	}  
}
```

然后配置一下这个Activity，让他能接收一个Action：

```xml
<activity  
    android:name=".EditActivity2"  
    android:exported="true"  
    android:label="@string/title_activity_edit2"  
    android:theme="@style/Theme.ActivityTest2"  
    android:launchMode="singleTask"  
    >  
    <intent-filter>        
	    <action android:name="com.example.activitytest2.ACTION_TEST" />  
        <category android:name="android.intent.category.DEFAULT" />  
    </intent-filter>
</activity>
```

```ad-warning
这里的exported一定要写成true！否则我们在其它应用中是没有权限启动它的！
```

然后，在AT1的代码中加上一个按钮，来启动这个Activity：

```kotlin
Button(onClick = {  
    val intent = Intent("com.example.activitytest2.ACTION_TEST")  
    context.startActivity(intent)  
}) {  
    Text(text = "Start EditActivity2")  
}
```

开始验证！我们依然分成两种情况。首先是**后台干干净净**：

![[Study Log/android_study/resources/scrcpy_03pSkdNpyk.gif|300]]

可以看到，从EditActivity2直接回到了AT1的MainActivity。接下来，是**启动AT2的情况下**：

![[Study Log/android_study/resources/scrcpy_CCvlUkRgd9.gif|300]]

**神奇的事情来了！启动了AT2之后，在中间多出来了一个AT2的MainActivity**。而这就是SingleTask的模式：在目标Task上新建出我想打开的Activity，然后把整个Task都压在我当前的Task上面。而这也就意味着，**如果之前目标的Task已经是启动状态的话，里面已经存在的Activity也回被顺带压过来**。图解就是这样的：

![[Study Log/android_study/resources/Drawing 2023-08-03 17.35.09.excalidraw.png]]

上图是启动了AT2之后进行操作的过程；而如果是后台干干净净的情况，AT2.MainActivity就不会存在了，甚至Task2也不会存在。

到了现在，我们能回答[[#^9b0410|之前那个问题]]了吗？答案是肯定的。想一想之前我们是怎么做的：启动一个Standard的Activity（此时这个Activity还是属于AT1），然后在这个上面启动了一个SingleTask的Activity。这就意味着如果之前AT2已经运行了，就会把AT2整个Task都搬过来，这样两个AT1中的AT2.MainActivity和AT2里面已经启动的AT2.MainActivity就贴在一起了：

![[Study Log/android_study/resources/Drawing 2023-08-03 18.03.39.excalidraw.png]]

而这就是中间出现了两次StartEdit的原因。下面，用一个动画来演示一下SingleTask启动的整个过程：

![[Study Log/android_study/resources/msedge_3hWkSszdz9.gif|500]]

**下面，我们进行一个，你可能从来没有意识到过的操作**。还是之前那个EditActivity2的例子。这次，在AT2运行的状态下进行操作：

1. 运行AT1；
2. 点击Start EditActivity2；
3. **上划，进入任务列表，然后再进入这个应用**；
4. 持续按返回键。

你会得到一个，完全出乎你意料的结果！

![[Study Log/android_study/resources/scrcpy_YmVmsrR0zp.gif|300]]

我的AT1.MainActivity呢？怎么没了？这涉及到前台Task和后台Task的问题。**我们正在运行的应用所在的Task，就是前台Task。而如果我们像[[Study Log/android_study/resources/Drawing 2023-08-03 18.03.39.excalidraw.png|刚才]]一样，使用SingleTask将Task进行了叠加，那么这多个Task共同作为前台Task**。

当我们进行以下操作时，前台Task会进入后台：

1. 按Home键进入桌面；
2. 进入最近任务列表。

```ad-warning
这里要注意第二条的**“进入”**二字。并不是切换到其它应用之后之前的前台Task才会进入后台，而是进入这个最近任务视图的一瞬间就切换到后台了。
```

而如果我们是那种多个Task叠在一起的情况，在进入后台时，**全部都会被拆开**。所以，才会导致上面AT1.MainActivity消失的情况。因为，那条表示了叠放关系的链子已经断了：

![[Study Log/android_study/resources/Drawing 2023-08-03 19.35.23.excalidraw.png]]


















[^错误的原因]: ComponentName和Action的方式，只是显式和隐式的区别。