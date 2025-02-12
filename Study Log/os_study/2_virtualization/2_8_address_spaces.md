---
title:
  - 2.8 Address Spaces
order: "9"
---
[[Study Log/os_study/0_ostep_index|Return to Index]]

## 2.8 Address Spaces

老式的OS，物理内存就是这样子的：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250212231356.png]]

操作系统就是一坨代码（其实就是一个库），从0k起始，一直到64k。然后剩下的就是一个正在运行的进程。老式的OS就是这么简单。

在然后，系统复杂了起来，进程更多了，还有了并发。这个时候，就需要OS有能力从一个进程切换到另一个进程，也就是我们之前介绍的时间片共享、LDE、调度之类的。此时，内存的使用就复杂了。每一个进程都需要一个自己的空间来存数据和代码。

这就需要我们让内存也能实现time sharing。有一种比较简单的方式。运行一个进程一段时间，让它能访问所有的内存，然后停止它，把进程所有的状态（也包括所有的物理内存）保存到磁盘中。加载其他进程的状态，运行一段时间……这样就是一个非常原始的共享内存机制。

[[Study Log/os_study/0_ostep_index|Return to Index]]