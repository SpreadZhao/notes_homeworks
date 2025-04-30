---
title: "2.10 Mechanism: Address Translation"
order: "11"
---
[[Study Log/os_study/0_ostep_index|Return to Index]]

## 2.10 Mechanism: Address Translation

在CPU的虚拟化中，我们介绍了LDE。在绝大多数情况下，让程序**直接**运行在硬件上，然而在一些关键点，比如程序发起了一个系统调用，或者一个时钟中断产生，安排操作系统入场，让正确的事情发生。就是这些时机，让操作系统能更好地控制程序，利用物理硬件。

其中一个OS要做的事情，就是内存虚拟化。

> [!question] 本章的问题：
> 1. 如何构建一个高效率的内存虚拟化？
> 2. 如何让程序用着方便灵活？
> 3. 如何控制程序能访问的内存，保证程序对内存的访问是受到限制的？
> 4. 如何让以上行为变得高效？

- [ ] #TODO tasktodo1746030134256 之前每章的CRUX要补一下，看看问题是否都能回答 ➕ 2025-05-01 🔺 🆔 cpraus 

我们要介绍的高科技是啥呢？标题写了，就叫地址翻译（Hardware-based Address Translation）。**硬件**会把每个指令的虚拟地址翻译成物理地址。

当然，也不能只靠硬件翻译。OS在特定的时机也要上场，追踪内存的使用情况才行。

现在我们来总结一下：程序有自己的私有内存，存放自己的数据和代码。在这背后是丑陋的物理层真相：程序们其实在共享同一片内存空间，CPU运行一个程序，然后运行下一个。通过虚拟化，操作系统在硬件的帮助下，可以把这个丑陋的真相做一个有用的、强大的、易用的抽象。

### 2.10.1 Assumptions

我们一开始做的假设很简单，甚至让你感觉可笑。别急，后面就是操作系统笑话你了：

1. 假设用户的地址空间在物理内存中必须是连续的；
2. 地址空间不能太大，比物理空间要小；
3. 每一个（程序的）地址空间需要是同样的大小。

### 2.10.2 An Example

下面这段代码：

```c
voic func() {
	int x = 3000;
	x = x + 3;
}
```

编译器把代码变成汇编，如果是x86汇编，就像下面这样：

```asm
128: movl 0x0(%ebx), %eax   ;load 0+ebx into eax 
132: addl $0x03, %eax       ;add 3 to eax register 
135: movl %eax, 0x0(%ebx)   ;store eax back to mem
```

上面三行是`x = x + 3`的汇编。x的地址现在在ebx寄存器，然后把这里面的值放到通用寄存器eax中，使用movl指令（用来移动longword）。下一行就是把eax的值增加3。最后一行是把eax中的值加载回ebx中，也就是回到内存（注意，这里是加载到ebx的地址上，所以是内存地址里，并不是放到ebx寄存器上。这个寄存器存的是地址不是值）。

- [ ] #TODO tasktodo1746033660626 有时间自己把这个汇编生成一下。 ➕ 2025-05-01 🔽 🆔 0zxcta 

在上面的例子中，内存情况如下：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250501010725.png|200]]

代码存在Program Code，从128地址开始（离0地址很近）。而变量x的值是3000,存在地址15KB。

当这些执行执行时，

[[Study Log/os_study/0_ostep_index|Return to Index]]