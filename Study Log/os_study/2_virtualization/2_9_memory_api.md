---
title:
  - Memory API
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



[[Study Log/os_study/0_ostep_index|Return to Index]]