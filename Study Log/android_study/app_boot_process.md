# 2 应用启动流程

[Android启动过程分析(图+文)-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1356506)

从点击桌面的APP图标，到APP主页显示出来，大致会经过以下流程：

1.  点击桌面App图标，Launcher进程采用Binder跨进程机制向system_server进程发起startActivity请求；
2.  system_server进程接收到请求后，向Zygote进程发送创建进程的请求，Zygote进程fork出新的子进程，即新启动的App进程；
3.  App进程，通过Binder机制向sytem_server进程发起attachApplication请求（绑定Application）；
4.  system_server进程在收到请求后，进行一系列准备工作后，再通过binder机制向App进程发送scheduleLaunchActivity请求；
5.  App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息。主线程在收到Message后，通过发射机制创建目标Activity，并回调Activity.onCreate()/onStart()/onResume()等方法，经过UI渲染结束后便可以看到App的主界面。

![[Study Log/android_study/resources/Pasted image 20230714144749.png]]

一些基础知识：

-   冷启动：当启动应用时，后台没有该应用的进程，这时系统会重新创建一个新的进程分配给该应用，这个启动方式就是冷启动，下文讲述的APP启动流程属于冷启动；
-   热启动：当启动应用时，后台已有该应用的进程（例：按back键、home键，应用虽然会退出，但是该应用的进程是依然会保留在后台，可进入任务列表查看），所以在已有进程的情况下，这种启动会从已有的进程中来启动应用，这个方式叫热启动；
-   一个APP就是一个单独的进程，对应一个单独的Dalvik虚拟机；[Android Runtime (ART) 和 Dalvik  |  Android 开源项目  |  Android Open Source Project](https://source.android.com/docs/core/runtime?hl=zh-cn)
-   Launcher：我们打开手机桌面，手机桌面其实就是一个系统应用程序，这个应用程序就叫做“Launcher”。同样的，下拉菜单其实也是一个应用程序，叫做“SystemUI”；
-   Binder：跨进程通讯的一种方式。
-   Zygote：Android系统基于Linux内核，当Linux内核加载后会启动一个叫“init”的进程，并fork出“Zygote”进程。Zygote意为“受精卵”，无论是系统服务进程，如ActivityManagerService、PackageManagerService、WindowManagerService等等，还是用户启动的APP进程，都是由Zygote进程fork出来的；
-   system_server：系统服务进程，也是Zygote进程fork出来的。该进程和Zygote进程是Android系统中最重要的两个进程，系统服务ActivityManagerService、PackageManagerService、WindowManagerService等等都是在system_server中启动的；
-   ActivityManagerService：活动管理服务，简称AMS，负责系统中所有的Activity的管理；
-   App与AMS通过Binder进行跨进程通信，AMS与Zygote通过Socket进行跨进程通信；
-   Instrumentation：主要用来监控应用程序和系统的交互，是完成对Application和Activity初始化和生命周期的工具类。每个Activity都持有Instrumentation对象的一个引用，但是整个进程只会存在一个Instrumentation对象；
-   ActivityThread：依赖于UI线程，ActivityThread不是指一个线程，而是运行在主线程的一个对象。App和AMS是通过Binder传递信息的，那么ActivityThread就是专门与AMS的外交工作的。ActivityThread是APP的真正入口，APP启动后从ActivityThread的main()函数开始运行；
-   ActivityStack：Activity在AMS的栈管理，用来记录经启动的Activity的先后关系，状态信息等。通过ActivtyStack决定是否需要启动新的进程；
-   ApplicationThread：是ActivityThread的内部类，是ActivityThread和ActivityManagerServie交互的中间桥梁。在ActivityManagerSevice需要管理相关Application中的Activity的生命周期时，通过ApplicationThread的代理对象与ActivityThread通信；

## 2.1 startActivity请求

系统启动过程中，会启动PMS服务，该服务会扫描解析系统中所有APP的AndroidManifest文件，在Launcher应用启动后，会将每个APP的图标和相关启动信息封装在一起。回想平时应用开发中，启动一个新的Activity是通过startAcitvity()方法。因此在桌面点击应用图标，也是在Luancher这个应用程序里面根据当前点击的APP的启动信息，执行startAcitvity()方法，通过Binder通信，最后调用ActivityManagerService的startActivity()方法。流程图如下：

![[Study Log/resources/Pasted image 20230704161238.png]]

## 2.2 Zygote fork新进程

![[Study Log/resources/Pasted image 20230704164058.png]]

当执行到startSpecificActivityLocked()方法时，会进行一次判断：

* 如果当前的程序已经有正在运行的Application，那么直接执行startActivity()即可；
* 如果并没有关联的进程，那么需要通过socket通道传递给Zygote进程，让它fork初一个新的进程来绑定这个Activity。

## 2.3 绑定Application

ActivityThread的main函数主要执行两件事情：

-   依次调用Looper.prepareLoop()和Looper.loop()来开启消息循环；
-   调用attach()方法，将给该新建的APP进程和指定的Application绑定起来；

## 2.4 启动Activity

经过2.2\~2.3后，系统就有了当前新APP的进程了，接下里的调用顺序就相当于从一个已经存在的进程中启动一个新的Actvity。因此回到2.2，如果当前Activity所在的Application有运行的话，就会执行realStartActivityLocked()方法，并调用scheduleLaunchActivity();

scheduleLaunchActivity()发送一个LAUNCH_ACTIVITY消息到消息队列中, 通过 handleLaunchActivity()来处理该消息。在 handleLaunchActivity()通过performLaunchActiivty()方法回调Activity的onCreate()方法和onStart()方法，然后通过handleResumeActivity()方法，回调Activity的onResume()方法，最终显示Activity界面。

![[Study Log/resources/Pasted image 20230704164706.png]]