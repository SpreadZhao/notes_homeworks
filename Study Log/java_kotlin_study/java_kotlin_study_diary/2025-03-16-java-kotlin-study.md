---
title:
  - Singleton
date: 2025-03-16
tags:
  - language/coding/java
mtrace:
  - 2025-03-16
---

# Singleton

单例模式，找到一篇比较全的文章：[Java Singleton Design Pattern Best Practices with Examples | DigitalOcean](https://www.digitalocean.com/community/tutorials/java-singleton-design-pattern-best-practices-examples)

## 1 Eager Initialization

饿汉式单例模式，只要类被加载了，就会被创建。因此缺点也很明显，不是随用随创建的：

```java
public class EagerInitializedSingleton {

    private static final EagerInitializedSingleton instance = new EagerInitializedSingleton();

    // private constructor to avoid client applications using the constructor
    private EagerInitializedSingleton(){}

    public static EagerInitializedSingleton getInstance() {
        return instance;
    }
}
```

## 2 Static Block Initialization

和[[#1 Eager Initialization]]唯一的区别就是，初始化放到了static块里，可以加一些额外逻辑，比如new对象的时候出现异常等等。缺点也是一样的：

```java
public class StaticBlockSingleton {

    private static StaticBlockSingleton instance;

    private StaticBlockSingleton(){}

    // static block initialization for exception handling
    static {
        try {
            instance = new StaticBlockSingleton();
        } catch (Exception e) {
            throw new RuntimeException("Exception occurred in creating singleton instance");
        }
    }

    public static StaticBlockSingleton getInstance() {
        return instance;
    }
}
```

## 3 Lazy Initialization

最原始的懒加载，虽说是随用随创建了，但是缺点也很明显，多线程的时候就出问题了：

```java
public class LazyInitializedSingleton {

    private static LazyInitializedSingleton instance;

    private LazyInitializedSingleton(){}

    public static LazyInitializedSingleton getInstance() {
        if (instance == null) {
            instance = new LazyInitializedSingleton();
        }
        return instance;
    }
}
```

## 4 Thread-safe Singleton

[[#3 Lazy Initialization]]的问题咋办？很简单！把getInstance方法标记成synchronized就可以了：

```java
public class ThreadSafeSingleton {

    private static ThreadSafeSingleton instance;

    private ThreadSafeSingleton(){}

    public static synchronized ThreadSafeSingleton getInstance() {
        if (instance == null) {
            instance = new ThreadSafeSingleton();
        }
        return instance;
    }

}
```

上面这个写法就已经能保证线程安全了，但是又带来一个问题：性能。如果在一开始有非常多的，比如n个线程同时调用getInstance，会让n-1个线程都阻塞在这个synchronized上。这就导致在**这个单例刚刚被使用的一小段时间内**，性能下降。

怎么解决？就是经常提到的double-check：

```java
public static ThreadSafeSingleton getInstanceUsingDoubleLocking() {
    if (instance == null) {
        synchronized (ThreadSafeSingleton.class) {
            if (instance == null) {
                instance = new ThreadSafeSingleton();
            }
        }
    }
    return instance;
}
```

这样，大部分线程会被外面的if给拦住，就不用走synchronized浪费时间了；少部分线程会走进去，但是是少部分，所以性能会好一些。

不过，你应该记得，这个实现里的instance好像是volatile的。为什么呢？下面来介绍。上面的实现在极端情况还是可能会出问题。参考：[3、双重校验锁【推荐】](https://blog.csdn.net/fly910905/article/details/79286680#t2)。

我们在[[Study Log/java_kotlin_study/concurrency_art/3_3_sequential_consistency#3.3.3 同步程序的顺序一致性效果|3_3_sequential_consistency]]中介绍过，[[Study Log/java_kotlin_study/concurrency_art/3_3_sequential_consistency#^0578f5|synchronized块中的代码，其实是可以重排序的]]。所以，可能会出现下面的情况：

1. 线程1调用getInstance，获得synchronized锁，进行instance赋值；
2. 在synchronized内部进行了重排序，**先把地址给到了instance，然后才new的对象**；
3. 线程2调用getInstance，因为是第一次调用，**线程2的==本地内存💬==中并没有instance的缓存，所以直接从主存中读**，就读到了线程1刚给分配的instance的地址。但是此时instance还没初始化呢；
4. 线程2使用instance做事情，读取里面的数据，发现啥也读不到，程序出现错误。

> [!comment] 本地内存
> 这里的本地内存概念见[[Study Log/java_kotlin_study/concurrency_art/3_1_JMM_basic|3_1_JMM_basic]]

那咋办？我们仔细思考一下，就能得出一个解决办法：只要保证instance在写入地址的时候，对象一定是初始化好的，那么就能避免这个问题。

咋避免？本质我们想一想，不就是：**不要让之前的【初始化逻辑】，被重排序到【对instance写入】的后面去吗**！这个时候，就不得不提到我们的volatile写了：

> 我们来观察一下这个表格，得出一些结论。首先是最后一列：
> 
> ![[Study Log/java_kotlin_study/concurrency_art/resources/Pasted image 20231119181250.png]]
> 
> 这一列意味着，如果第二个指令为对volatile的写，那么不管第一个指令是什么都不允许重排序。本质是什么？本质就是，**你不能把对volatile的写指令往前挪**！

上面的内容出自[[Study Log/java_kotlin_study/concurrency_art/3_4_volatile_mm_semantics#3.4.3 volatile内存语义的实现|3_4_volatile_mm_semantics]]。显然，如果instance是volatile的，那么在它写入之前，所有的代码都必须执行完才行。在单例模式中，也就是当`instance = xxx`执行的时候，这个instance的new必须已经执行完毕了。

这样，就保证了这个单例模式彻底是Thread-safe的了。

## 5 Bill Pugh Singleton Implementation

volatile是java5引入的。因此在这之前上面的double-check方法是没法用的。那咋办？跳表的发明人[Bill Pugh](https://en.wikipedia.org/wiki/William_Pugh_(computer_scientist))发明了一种新的单例模式写法，没有并发控制，但是也是随用随创建的：

```java
public class BillPughSingleton {

    private BillPughSingleton(){}

    private static class SingletonHelper {
        private static final BillPughSingleton INSTANCE = new BillPughSingleton();
    }

    public static BillPughSingleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```

它用了一个静态内部类持有实例。在这个外面的类被加载的时候，里面的静态内部类并不会加载到内存中。只有有人调用了getInstance的时候，这个内部类才会被创建，从而创建单例类的实例。

剩下的内容就不太重要了，主要介绍了：

- 使用反射来破坏单例模式，也就是勾出来构造方法再new一个；
- 使用枚举来避免反射的破坏，因为一个java程序里只能有一个枚举，但是这样的话又不是lazy的了；
- 单例模式实现序列化的问题，从文件中读的时候会再new一个，避免这个需要重新实现readResolve()。