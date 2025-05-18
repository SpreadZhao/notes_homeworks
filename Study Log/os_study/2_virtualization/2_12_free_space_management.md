---
title: 2.12 Free-Space Management
order: "13"
---
[[Study Log/os_study/0_ostep_index|Return to Index]]

## 2.12 Free-Space Management

本章讨论的内容不是内存虚拟化的。而是更加普遍的问题，内存管理的问题，叫做free-space management。

在上一章我们讨论过，Segmentation会有External Fragmentation的问题：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250518171206.png|300]]

在这个例子中，如果我们要申请15Byte的空间，那是会失败的，即使空闲空间有20Byte。

我们能发现，这个问题产生的一个很大的原因是：**我们管理内存的具体单元，是一个一个字节，并且进程在申请内存的时候，申请的空间大小也都是不同的**。书中管这个叫variable-sized units。而Segmentation就是这样的管理方式。正是这种方式导致了External Fragmentation问题的产生。

因此，提出本章的CRUX：

> [!question] 如何管理空闲内存？
> 1. 如何管理空闲内存，还要能适配不同大小的请求（variable-sized requests）？
> 2. 用什么办法能最小化Fragmentation问题？
> 3. 如果用新的办法，那么新的办法的时间/空间开销是多少？

### 2.12.1 Assumptions

我们还是从`malloc()`和`free()`说起。[[Study Log/os_study/2_virtualization/2_9_memory_api#2.9.3 free|之前]]已经有介绍过了，`free()`传入的参数只有一个指针，并没有传大小。所以，这个库肯定能知道这个指针指向的空间是多大。我们就从这个问题作为切入。

管理内存的这个库，负责管理的肯定是堆内存。而堆内存中的一个普遍的数据结构叫做free list，包含了所管理的内存空间中的所有空间内存的引用。当然，这个free list本质上不一定要是个list，只要能追踪空闲内存就可以了。

还有，我们之前讨论过的External Fragmentation和Internal Fragmentation。前者被浪费的空间不属于任何进程，是空闲空间；后者浪费的空间是属于一个特定的进程的，因为分配器给它分配多了一点儿，但是它又没有使用，所以被浪费了。我们本次主要聚焦于External。

我们需要假设一件事情，为了讨论起来更容易，就是，一旦内存被分配给了一个进程，那么在调用`free()`之前，操作系统是不能进行任何==分配和改动==。

> [!comment] 分配和改动
> 这里我详细解释一下“分配和改动到底是什么”。比如给一个进程分配了空间，从100-105，物理地址。那么这个时候，这6个字节就属于这个进程了，不能再分配给别人。在实际的操作系统中，100-105很有可能在这个过程中（调用`free()`之前）被分配给其它进程，比如，在我们之前介绍的Segment实现中，会有[[Study Log/os_study/2_virtualization/resources/Pasted image 20250511212934.png|Compaction]]操作。在空间不足的时候，100-105很有可能就不再属于这个进程了，此时可能变成了400-405或者任何其它的地址。同时，100-105也可能（部分或全部）被重新分配给其它进程使用。上述的这个操作就是一种“分配和改动”。我们假设这种事情不会发生。
> 
> 这里再提一下Compaction的题外话。C语言中，指针是很灵活，很危险的。比如一个`void *`，甚至`int *`类型的指针，你取出这个值，比如`int *p; xxx; int x = *p;`，这个时候，`*p`指向的空间是4个字节没错，编译器认为这块空间中的内容是一个整数也没错。但是否真的是这样，可不一定。比如，我完全可以用一个`void *`的指针，访问到这个`p`指向的空间（4个字节）中的某一个字节，然后修改成一个奇怪的东西。比如调用`memset()`就可以做到。好像是把。那既然如此，我们可以发现，C语言是**弱类型**的，也就是说，类型只是个给程序员看的标记，本质上还是内存而已。而在一些**强类型**的语言中，比如java，类型就是一个很明确的概念，我们没办法钻到一个类型引用指向的内存里面，随意修改里面的数据。因此，在这些强类型的语言中，尤其是有GC的语言中，会通过Compaction来进行内存优化，因为GC是负责保管所有引用的，它知道所有引用。

最后，我们假设我们管理的内存空间大小是固定的，也就是说，咩有`brk()`、`sbrk()`这种系统调用。

### 2.12.2 Low-level Mechanisms



[[Study Log/os_study/0_ostep_index|Return to Index]]