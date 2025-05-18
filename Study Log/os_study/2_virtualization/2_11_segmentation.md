---
title: 2.11 Segmentation
order: "12"
---
[[Study Log/os_study/0_ostep_index|Return to Index]]

## 2.11 Segmentation

> 说实话，写笔记的时候，我其实已经知道什么TLB，分页之类的概念了。而到目前为止，其实这些概念都还没出现过，对内存的处理还是按一个地址（字节）一个地址来的。

首先，看看之前那张虚拟内存空间的图：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250214004408.png|400]]

这张图里，中间有一大片空闲的地方。在我们现在的策略下，这块区域是被浪费的。因为它只能由对应的进程使用，同时这个进程并没有使用。这种浪费叫做internal fragmentation，因为是inside的内存被浪费掉。这样，如果我们想要提供比较大的地址空间给每个进程，就非常困难。 ^78662f

本章的CRUX：

> [!question] 如何提供一个宽敞的地址空间
> 1. 如何提供一个大大的地址空间，同时栈和堆中间还能有足够的空闲空间？
> 2. 如果你的内存一共4GB，一个进程只使用了几MB，但是它就是要4GB内存。这个时候咋办？

### 2.11.1 Segmentation: Generalized Base/Bounds

为了解决浪费的问题，产生了一个办法，就叫做**segmentation**。这个其实是一个很老的概念了。

思想很简单：如果你给程序分成几份，每一份有一对base register \& bounds register，那不就没问题了？

上面的图中，浪费的地方就是栈和堆中间的部分。那么，如果我给一个进程的代码空间、栈空间、堆空间都分成不同的**段**，然后在物理内存中，安排一个个**段**，而不是进程。这样不是就不会有浪费问题了？在这种情况下，那些没有使用的内存就可以继续被其它进程使用，不会造成浪费。

> [!note]
> 这里提到的段，就是我们常说的“数据段”，“代码段”，“栈段”。

下面是一个例子。

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250504005156.png|200]] ![[Study Log/os_study/2_virtualization/resources/Pasted image 20250504004706.png|200]]

这个图还是比较好看懂的。注意Code是固定大小，所以不会增长，而对栈可以增长，所以有个箭头，也预留了一大堆空间。

下面的问题是，我们怎么做地址翻译呢？现在，我们应该就要有三对寄存器给这个进程了：

| Segment | Base | Size |
| ------- | ---- | ---- |
| Code    | 32K  | 2K   |
| Heap    | 34K  | 3K   |
| Stack   | 28K  | 2K   |

^30bc4e

下面，我们通过虚拟地址翻译物理地址就很简单了。这里需要注意的是，虚拟地址也不是从0开始。比如堆空间的虚拟地址是从4KB开始的。所以虚拟地址4KB对应物理地址34KB。

当访问了不该访问的地址时，通过一样的手段也能检测出来并移交给操作系统处理。这里，其实就是我们经常遇到的：**Segmentation Fault**。

> [!tip] 书上的一个关于Segmentation Fault的小玩笑
> Humorously, the term persists, even on machines with no support for segmentation at all. Or not so humorously, if you can’t figure out why your code keeps faulting.

### 2.11.2 Which Segment Are We Referring To?

还是上面的例子，如果我们要访问虚拟地址4200，是啥？**可以看到**，4200是4KB之后的一段地方，所以大概应该是堆空间。而偏移量就是$4200 - 4KB = 104$。所以，物理地址应该是$34KB + 104 = 34920$。

那么问题来了，我也加粗了，“可以看到”。那计算机咋看到？所以，我们要设计一些策略，让计算机能根据虚拟地址找到对应的segment，找到这个segment对应的起始地址，算出偏移量。

第一种解法非常暴力：用虚拟地址的前面几位当作段的index。

现在有三个段，所以我们需要2个bit来表示（为啥不用我说了吧。。。）。然后如果虚拟地址是14bit的（为啥呢？因为VAX/VMS操作系统就是这样），我们的虚拟地址就像这样：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250504012206.png|400]]

这样：

- 以00开头的地址，就是代码段的；
- 以01开头的地址，就是堆段；
- 以10开头的地址，就是栈段。

现在再来翻译一下之前的4200吧。4200的二进制是这样的：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250504012358.png|400]]

以01开头，所以是堆空间的。然后，更巧的是，我们看看后12bit。竟然正好是104，就是我们刚刚算过的偏移量！

咋会这么巧？当然这是必然的。因为01开头的都是堆空间，所以01000...0就是堆空间的起始地址。算一下，正好就是4KB。

然后，00，01，10也正好就是0,1,2。所以，可以用数组（一个base数组，一个bounds数组）来存这些segment。下面是翻译这个14bit地址的伪代码：

```c
#define SEG_MASK    0x3000
#define SEG_SHIFT   12
#define OFFSET_MASK 0xFFF

// get top 2 bits of 14-bit VA
segment = (virtual_address & SEG_MASK) >> SEG_SHIFT;
// now get offset
offset = virtual_address & OFFSET_MASK
if (offset >= bounds[segment]) {
	raise_exception(PROTECTION_FAULT);
} else {
	phys_addr = base[segment] + offset;
	register = access_memory(phys_addr);
}
```

下面是这个方案的缺点。很显然，11没用上啊也。所以，为了节省空间，某些系统会只用一个bit来表示段。那仨段咋办？把code和heap放一起。毕竟code是固定大小的。

下一个问题就是，你用了bit去表示段，那剩下能表示的地址就少了。换句话说，每个segment的虚拟空间都会变小。

还有一种从侧面推测出地址属于哪个segment的方法，这部分就放原文了：

> If, for example, the address was generated from the program counter (i.e., it was an instruction fetch), then the address is within the code segment; if the address is based off of the stack or base pointer, it must be in the stack segment; any other address must be in the heap.

### 2.11.3 What About The Stack?

最后是栈空间。我们把它放到最后说，因为它最特殊。特殊在哪儿？方向是反的。再贴一遍上面的图：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250504005156.png|200]] ![[Study Log/os_study/2_virtualization/resources/Pasted image 20250504004706.png|200]]

栈空间从==28KB开始==，缩缩到26KB。对应着虚拟地址的16KB到14KB。在这种情况下，我们应该怎么翻译地址呢？

> [!comment]- 从28KB开始 & 空间的末尾
> 这里的开始其实应该是要加引号的。因为栈空间的最后一个地址并不是28KB，而应该是28KB - 1。具体原因相信不用我多说。我们规定每一个空间的起始地址要包括进去，所以，假设起始地址是5，总大小是4，所以这个空间的所有地址就应该是5,6,7,8。而下一个空间的起始地址才是5 + 4 = 9。这样的规定就是为了计算方便。另外提示一下，这一切的原因是因为我们从0地址开始。

这里回头看一下我们上面给三个寄存器的[[#^30bc4e|赋值]]。可以看到，栈空间的Base Register里存的是28KB，是==空间的末尾==。所以，不能像处理堆空间一样处理它了。真实的地址应该是28KB减掉某一个值。

这就意味着，我们还需要一个信息，来表示这段空间的翻译应该是加还是减。换句话说，**我们要知道空间增长的方向**：

| Segment     | Base | Size (max 4K) | Grows Positive? |
| ----------- | ---- | ------------- | --------------- |
| $Code_{00}$ | 32K  | 2K            | 1               |
| $Heap_{01}$ | 34K  | 3K            | 1               |
| $Stack{11}$ | 28K  | 2K            | 0               |

我们把上面的表格更新了一下。可以看到，除了Base Register和Bounds Register，又多了3个bit。它表示增长的方向。有了这个信息，我们翻译起来就舒服了。

假设我们要翻译虚拟地址15KB，把它写成二进制：

```
11 1100 0000 0000
```

是11开头的，所以是栈空间。然后，后面剩下的bit是3KB。这个时候就出现问题了：它是啥的偏移量？

在上面heap翻译4200的例子中，我们取出偏移量，是基于4KB的偏移量。那么在现在stack的例子中，是基于谁的？显然，是基于：

```
11 0000 0000 0000
```

也就是12KB的。那么我们现在的策略是：获得当前虚拟地址距离当前segment末尾的距离，然后用物理地址的Base Register减去这个距离就可以了。

首先是虚拟地址的距离，咋算呢？想咋算咋算，信息都是全的。我们这里最快的做法，是用$3KB - 4KB$。你会问4KB是啥，答案是，**一个segment允许的最大容量**。为啥是4KB？因为我们去掉了前两个bit，所以，前两个bit定住加上后面所有的组合，就是一个segment的所有地址范围。具体就不说了，反正是4KB。然后，这样一减，就i知道了，从Stack的末尾往回缩缩1KB，就是我们的虚拟地址。

最后，让物理末尾28KB也往回缩缩，就得到了最终的虚拟地址27KB。

### 2.11.4 Support for Sharing

这一节说的是，让我们现在这个Segment机制支持一些牛逼的功能，比如内存共享。在当今的程序中，代码共享（code sharing）几乎无处不在。而这就可以通过，在物理内存中共用一个Code Segment来实现。

我们再来看一下内存，可以简单分成“可读”，“可写”，“可执行”。所以，我们可以用3个bit来表示这些，然后分别给不同的segment设置上。这样，对于那些read-only的segment，OS就可以偷偷在内部共用同一片内存，而不被上层的进程发现，这些进程依旧认为，它们访问的代码段是自己私有的虚拟地址空间的。

| Segment     | Base | Size (max 4K) | Grows Positive? | Protection |
| ----------- | ---- | ------------- | --------------- | ---------- |
| $Code_{00}$ | 32K  | 2K            | 1               | `r-x`      |
| $Heap_{01}$ | 34K  | 3K            | 1               | `rw-`      |
| $Stack{11}$ | 28K  | 2K            | 0               | `rw-`      |

### 2.11.5 Fine-grained vs. Coarse-graind Segmentation

这两个词是细粒度和粗粒度。我们上面讨论的segment的分法，只分成了栈、堆、代码。是粗粒度。在一些老系统上，还有细粒度的分法。当然细粒度需要更多的硬件支持。更多的就看书吧。

### 2.11.6 OS Support

这里讲了三个，在Segment机制下，操作系统相关的三个问题。

1. 在Context Switch的时候，操作系统需要对Segment做什么？

这个问题的答案你应该是要知道的。如果不知道，那说明学的还不够深刻。答案就在[[Study Log/os_study/2_virtualization/2_10_address_translation#2.10.5 Operating System Issues|2_10_address_translation]]，**保存和恢复每个segment的寄存器**。

2. 当Segment增长或回缩的时候，操作系统需要做什么？

比如，一个进程调用了`malloc()`来获取内存，有可能已经分配的堆空间就能够满足要求，那么直接分配上，返回指针就行了；但是有可能已经有的空间不够了，这个时候，就需要**让堆增长**。比如通过`sbrk()`系统调用，这样就能额外增加一些空间了。当实在不能再分配的时候（比如上面的例子里已经分配满了4KB），那就会返回失败了。

3. 最重要的，怎么管理空闲内存？

我们现在的Segment机制，有一个很大的问题，叫做External Fragmentation。和之前的[[#^78662f|Internal Fragmentation]]相对，显然这是程序之间的浪费。产生原因也很简单，因为实际情况下，每个进程的Segment大小都多多少少不一样，所以，分配在物理内存中的时候，就难免出现很多小的空闲空间，就跟一堆窟窿一样，谁也用不了（因为太小了）。

一个粗暴的解决办法如下图：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250511212934.png|400]]

在程序没有运行的时候（被切走），把它的内存都copy到一段连续的内存中，然后修改各种寄存器。这样空间就腾出来了。但是，这样做的缺点也很明显：

1. 性能很垃圾，因为整理一次内存不是那么简单的工作，程序如果必须等待整理完成才能继续运行，那么性能就太拉了；
2. 这种情况下，如果segment想增长咋办？都堆在一块了，很可能出现一个进程的heap和另一个进程的stack顶到一起去了。这种情况下，就只能再重新分配，然后又引发External Fragmentation，反反复复。

下面是一个更加轻量的解决方式，也就是通过让分配的内存尽可能更加合理。前面提到过我们用free list来追踪内存使用情况，而在分配内存的时候，就可以用一个更加巧妙的算法来实现。

具体的算法太多了，什么best-fit，worst-fit，first-fit等。这些我在上OS课的时候有记过笔记，这里就不多说了。不过，不管什么方式，都不能完全解决External Fragmentation，只能尽量避免。你用再好的分配方式，都可能有一两个字节被空出来，那就会产生这个问题。

本章结束了，最后还有一个最大的问题，是关于Segment的缺陷的：

举个例子，如果我们有一个非常大的堆，但是没咋用。这个大堆依然要整个放在物理内存中。而且Segment还是连续的，都堆在一块儿。就拿我们上面翻译的例子来说，都是base + offset。那就代表这个Segment必须是一段连续的地址，就非常不灵活。如果一个4MB的大堆哐一下子怼在内存里，然后你就用了一个字节。剩下的空间其它进程还是用不了的。总之，还是不灵活。

怎么解决这个问题？请听下回分解。~~你妈的我知道是page。~~

[[Study Log/os_study/0_ostep_index|Return to Index]]