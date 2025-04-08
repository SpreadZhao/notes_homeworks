---
title:
  - "2.9 Interlude: Memory API"
order: "10"
---
[[Study Log/os_study/0_ostep_index|Return to Index]]

## 2.9 Memory API

这一章的内容很确定：介绍UNIX是怎么分配内存的。

### 2.9.1 Types of Memory

我已经很熟了，栈内存和堆内存。重新回顾一遍吧。

**Stack Mem**是我们在函数里定义变量的时候，编译器**自动**帮我们分配好的。所以栈内存有时候也叫automatic mem。比如：

```c
void func() {
	int x;    // declares an integer on the stack
}
```

定义x的时候分配内存，当func返回的时候，这个内存就被自动释放了。所以，如果你需要一些函数外使用的内存，它们不应该被分配在栈上。

相对的，我们自己分配的，不是自动的，可以在函数外使用的内存，就是heap mem。

```c
void func() {
	int *x = (int *) malloc(sizeof(int));
}
```

这里有个小问题：两种内存的分配都出现了。堆内存是我们调用malloc申请的。同时，这里也有栈内存的分配。就是分配给x这个指针了。这里x保存的，其实就是malloc函数的返回值，它是`int *`类型的。

### 2.9.2 malloc

malloc位于`stdlib.h`。你可能还看过`malloc.h`，但是，这个已经deprecated了：[The &lt;malloc.h&gt; heade](https://stackoverflow.com/questions/12973311/difference-between-stdlib-h-and-malloc-h)。但实际上什么头文件都不加，就可以用malloc了， 因为所有的c程序本身就会默认添加这个头文件。加上这个头文件的意义是让编译器查看你是否正确调用了malloc。比如传参是否正确等等。

> [!comment]- 不加头文件编译会报这样的错误
> 
> ~~~bash
> ❯ make
> gcc malloc.c -o ../out/malloc
> malloc.c: In function ‘main’:
> malloc.c:4:22: error: implicit declaration of function ‘malloc’ [-Wimplicit-function-declaration]
>     4 |     int *a = (int *) malloc(sizeof(int));
>       |                      ^~~~~~
> malloc.c:2:1: note: include ‘<stdlib.h>’ or provide a declaration of ‘malloc’
>     1 | #include <stdio.h>
>   +++ |+#include <stdlib.h>
>     2 | 
> malloc.c:4:22: warning: incompatible implicit declaration of built-in function ‘malloc’ [-Wbuiltin-declaration-mismatch]
>     4 |     int *a = (int *) malloc(sizeof(int));
>       |                      ^~~~~~
> malloc.c:4:22: note: include ‘<stdlib.h>’ or provide a declaration of ‘malloc’
> make: *** [makefile:4: malloc] Error 1
> ~~~

malloc只有一个参数，类型是`size_t`。就是表示你需要多少字节。通常，我们会传入一个特定类型的表达式。比如`sizeof(double)`。这是一个编译器就能确定的操作符，在编译之后这块直接就变成8了。

`sizeof`也可以直接传变量。但是返回的结果可能和我们想像的不一样：

```c
#include <stdlib.h>
#include <stdio.h>

int main() {
    int *x = malloc(10 * sizeof(int));
    printf("size of int: %d\n", sizeof(int));
    printf("%d\n", sizeof(x));
}
```

最后一行我们传入的就是变量x。那么x的大小是多少呢？是40吗？不是。答案是4或者8，取决于我们是32位还是64位。我们分配了一个10个int的数组，但是最后存储的变量是x，它的类型是`int *`。所以最后输出的就是`int *`的大小。

那问题就是，为啥编译器不能知道我们要分配40，而是只输出了`int *`的大小呢？答案就在我们刚刚提到的动态。堆内存是动态分配的，~~所以只有运行起来之后，才知道这个位置传的是多少~~在运行起来之后，malloc将内存分配好，然后把分配的内存的首地址返回给x。这里就在于，我们给sizeof的只有一个x，也就是首地址。这个内存到底是多大，编译器不知道。它只关心x这个变量占了多大内存，那自然就是指针类型的大小，8个字节。

与之相对的，我们看这个例子：

```c
#include <stdlib.h>
#include <stdio.h>

int main() {
    int a[10];
    printf("size of a: %d\n", sizeof(a));
}
```

a分配在栈内存上，也是个数组。但是因为是栈内存，在这个函数调用完就返回了。所以编译器有足够的信心证明a的大小就是固定的10个int。所以在编译期就能够定下来，因此输出的就是40。

> [!note]- 更多sizeof的例子
> 
> [sizeof operator - cppreference.com](https://en.cppreference.com/w/cpp/language/sizeof)
>
> ---
> 
> 书上还介绍了一个好玩的。当我们给字符串分配内存的时候，比如我们已经有了一个字符串s，那么如果想分配一段能存下这个字符串的空间，应该用：
> 
> ```c
> malloc(strlen(s) + 1)
> ```
> 
> 而不是用：
> 
> ```c
> malloc(sizeof(s))
> ```
> 
> 为什么呢？我们都知道，sizeof比strlen多了一个尾部的`\0'`。那么，为啥用sizeof就不行呢？我找到了这篇文章：[Surprising results between sizeof() and strlen when allocating a string with the wrong number of elements : r/C_Programming](https://www.reddit.com/r/C_Programming/comments/ddqt10/surprising_results_between_sizeof_and_strlen_when/)
> 
> 下面的代码：
> 
> ```c
> char str1[5] = "hello";
> char str2[] = "hello";
> printf("str1: sizeof is %d, strlen is %d\n", sizeof(str1), strlen(str1));
> printf("str2: sizeof is %d, strlen is %d\n", sizeof(str2), strlen(str2));
> ```
> 
> 我的电脑输出是这样：
> 
> ```
> str1: sizeof is 5, strlen is 10
> str2: sizeof is 6, strlen is 5
> ```
> 
> 好吧，我感觉这个例子并没说明上面那个问题。这里strlen反而出错了。因为它的工作原理是找到字符串的最后一个`\0`。而str1没有`\0`，所以会继续往后找，导致大于5。 在strlen的man page中，也推荐我们在这种情况使用strnlen，有一个最大值。

然后，`malloc()`的返回值是`void *`类型。最后转换成什么由程序员来决定。

### 2.9.3 free

```c
int *x = malloc(10 * sizeof(int));
...
free(x);
```

这里要注意的是，free并没有要我们传入free的空间有多大。这是因为这一套内存分配库会自己追踪我们分配的内存。

### 2.9.4 Common Errors

使用Memory API时会犯的常见错误：

1. 忘记分配内存

下面的代码：

```c
char *src = "hello";
char *dst;
strcpy(dst, src);
```

会直接崩掉。因为dst还没有分配内存。

所以，正确的做法是：

```c
char *src = "hello";
char *dst = (char *) malloc(strlen(src) + 1);
strcpy(dst, src);
```

或者，也可以借助`strdup()`的力量：

```c
char *src = "hello";
char *dst = strdup(src);
```

malloc在不同平台，不同版本的实现细节都是不一样的。比如，某些情况下，在复制字符串的时候，会在分配空间的末尾再多分配一个字节（比如上面的例子，最后分配的就是`strlen(src) + 2`）。这样的操作一般来讲都是没问题的，也能留一些缓冲。而在另一些情况下，这甚至是系统漏洞的来源（“Survey on Buffer Overflow Attacks and Countermeasures” by T. Werthman. Available）。

2. 忘记初始化分配的内存

这个没啥好说的，分配了内存，总得往里面塞点值。

3. 忘记释放内存

这个就是引起内存泄漏的重要原因。

4. 内存用完之前就释放了

这种类型的指针叫做Dangling Pointer。

> [!note] 程序退出的时候，没有内存会泄漏
> 
> 假设你的代码很短，就上上面例子这样。你调用了malloc。那么程序退出之前需要调用free吗？看起来是需要的，但实际上没必要（还是调用更标准一些）。即使不调用，也不会出现内存泄漏。
> 
> 原因是，OS中其实有两套内存管理：一套是进程中自己的，包括堆内存，栈内存。也就是malloc，free管理的这些；另一套是操作系统自己管理的。在进程结束之后，不管进程自己有没有释放内存，这些内存都会被操作系统回收回来。注意，是所有内存。
> 
> 但是，如果是那种服务器程序，或者那种一直留在手机后台的Android程序，就可能出现这种内存一直留着没回收的状态。
> 
> - [ ] #TODO tasktodo1744127756776 有时间看看Android，JVM的内存管理是不是也是这样的。为啥Android程序内存老泄漏呢？handler被泄漏之后，也是这样的逻辑么？ ➕ 2025-04-08 🆔 ubf06h 🔽 

5. 重复释放内存
6. 错误调用free

比如我随便传了一个乱七八糟的东西：

```c
free((void *) 1);
```

这段代码在执行的时候就直接崩了。因为这个1是分配在栈上的，要释放一个栈上的内存，那自然就炸了。

### 2.9.5 Underlying OS Support

[[Study Log/os_study/0_ostep_index|Return to Index]]