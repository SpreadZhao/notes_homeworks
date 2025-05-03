---
title: 2.11 Segmentation
order: "12"
---
[[Study Log/os_study/0_ostep_index|Return to Index]]

## 2.11 Segmentation

> 说实话，写笔记的时候，我其实已经知道什么TLB，分页之类的概念了。而到目前为止，其实这些概念都还没出现过，对内存的处理还是按一个地址（字节）一个地址来的。

首先，看看之前那张虚拟内存空间的图：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250214004408.png|400]]

这张图里，中间有一大片空闲的地方。在我们现在的策略下，这块区域是被浪费的。因为它只能由对应的进程使用，同时这个进程并没有使用。这种浪费叫做internal fragmentation，因为是inside的内存被浪费掉。这样，如果我们想要提供比较大的地址空间给每个进程，就非常困难。

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

[[Study Log/os_study/0_ostep_index|Return to Index]]