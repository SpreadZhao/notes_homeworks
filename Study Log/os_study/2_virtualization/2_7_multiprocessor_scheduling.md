---
title:
  - 2.7 Multiprocessor Scheduling
order: "8"
---
[[Study Log/os_study/0_ostep_index|Return to Index]]

## 2.7 Multiprocessor Scheduling

> [!attention]
> 关键问题：操作系统是怎么在多线程处理器上做调度的？会出现什么新的问题？我们前面讲到的老办法还好使吗？还是需要一些新的idea？

### 2.7.1 Background: Multiprocessor Architecture

都知道，CPU是有Cache的：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250209235446.png]]

Cache中存放的是**popular**的数据（书上的这个描述太精辟了，所以我抄了下来）。而主存中存放所有的数据。Cache更快，但是小，只能存很小的一坨。用Cache可以加速内存访问。

Cache是基于**局部性**的。有两种：

- Temporal Locality：时间局部性。当一个数据被访问了，那么在不久的将来很有可能再次被访问。比如循环中不断递增的`i`；
- Spatial Locality：空间局部性。如果程序访问了一个位于x地址的数据，那么在x附近的数据很可能也会被访问。比如一个数组。

然后，就是老生常谈的缓存一致性（cache coherence）问题。这部分书上有个例子：

> Imagine, for example, that a program running on CPU 1 reads a data item (with value D) at address A; because the data is not in the cache on CPU 1, the system fetches it from main memory, and gets the value D. The program then modifies the value at address A, just updating its cache with the new value D′ ; writing the data through all the way to main memory is slow, so the system will (usually) do that later. Then assume the OS decides to stop running the program and move it to CPU 2. The program then re-reads the value at address A; there is no such data in CPU 2’s cache, and thus the system fetches the value from main memory, and gets the old value D instead of the correct value D′. Oops!

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250210001132.png]]

咋解决呢？有一个最简单的办法：总线窥探（bus snooping）。当CPU观察到，它的Cache中的数据有更新的时候，要么让这块数据无效，要么直接更新成新的。




[[Study Log/os_study/0_ostep_index|Return to Index]]