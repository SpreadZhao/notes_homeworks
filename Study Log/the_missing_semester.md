---
author: "Spread Zhao"
title: the_missing_semester
category: self_study_long
description: MIT学校的The Missing Section课程，能学到一些非常有用的编程知识和技巧
---

# 1. The Shell

## 1.1 echo

`echo`命令：将文本显示在终端。它被添加到环境变量中。

## 1.2 which

`which`命令：用于输出环境变量的某个文件的绝对路径。比如我安装了好多echo，想要知道用的是安装在哪里的echo，就只需要输入`which echo`。

## 1.3 escape

Escape：这个是之前没有想到的。比如我要创建一个目录叫做`"My Photos"`。但是因为中间有空格，所以如果我执行`mkdir My Photos`，这其实创建了两个目录。所以通常我的做法是`mkdir "My Photos"`。但是别忘了转义的作用，**如果我想表示空格只是一个空格，并不是分隔参数的符号**，那么我完全可以`mkdir My\ Photos`。这样其实也能达到一样的效果。

## 1.4 redirection

任何文件/程序打开时都默认会打开两个流：

* standard input stream - keyboard
* standard output stream - screen

所以我们可以通过任意组合来达到一些复杂的效果。比如我执行`echo hello`，这只是将hello这串字符显示在我的终端。如果我不想让它显示在(输出到)终端，而是输出到一个文本文件中，我们可以这样做：

```shell
echo haha > hello.txt
```

`>`这个符号可以**更改输出流**，这样输出的对象就不再是默认的屏幕，而是`hello.txt`这个文件了。

---

而如果我们想要更改输入流，自然是使用`<`符号。比如我们想要查看`hello.txt`中的内容，通常是：

```shell
cat hello.txt
```

但是我们还可以这样：

```shell
cat < hello.txt
```

这时发生的就是：shell打开hello.txt，读取其中的内容，作为cat程序的输入流；而cat从它的输入流拿到东西之后，就要把这些内容放到输出流。由于没有设置输出流，自然就是默认的屏幕了。所以我们会在终端看到其中的内容：

接下来，我们更改一下这个输出流，将输出的内容放在hello2.txt文件中：

```shell
cat < hello.txt > hello2.txt
```

这样我们就能够看到新建了一个hello2.txt文件，其中的内容正是haha。我们不难发现，这其实正是copy操作所做的事情。只不过我们这个程序只支持文本文件罢了。

#question 为啥这里我用`echo < hello.txt`就不能在屏幕上输出haha？理论上不是输入流从键盘变成了hello.txt这个文件吗？然后echo从它的输入流拿到了内容haha，然后显示在屏幕上？难道echo不能改输入流，只能用键盘？

---

另外还有一个重定向的符号：`>>`这个符号表示追加输出。比如我们在当前基础上执行：

```shell
cat < hello.txt >> hello2.txt
```

这时hello2.txt中的内容就会变成：

![[Study Log/resources/Pasted image 20221215225631.png]]

## 1.5 pipe

通常我们过滤结果的时候会这样：

```shell
cat haha.txt | grep hehe
```

![[Study Log/resources/Pasted image 20221215225739.png]]

这是找出haha.txt这个文件中包含hehe字符串的所有行。但是我们似乎从来没考虑过中间的这个`|`到底是什么。其实它就是pipe，管道的意思就是，**左边的输出是右边的输入**。因此，`cat haha.txt`的输出是文件中的所有字符，这些字符就作为grep程序的输入，然后grep程序会从输入中过滤出我们想要的东西。我们可以再通过几个例子来看看：

```shell
ls -l | tail -n1
```

![[Study Log/resources/Pasted image 20221215225900.png]]

`ls -l`列出当前文件夹下的文件，而`tail`表示只是输出最后几行，而`-n1`表示输出最后一行。所以如果我们想要查看haha.txt文件中的最后两行文本，就可以这样写：

```shell
cat haha.txt | tail -n2
```

![[Study Log/resources/Pasted image 20221215225940.png]]

从这里我们也能发现，**pipe其实就是帮助我们重定向了这些程序的输入流和输出流**。

## 1.6 sudo, tee

有时候，即使我们在前面加上了`sudo`，还是会得到Permission denied的错误。这是为什么呢？我们从一个例子来看看。`/sys/class/backlight/`目录里存的是屏幕亮度。如果我们打算修改这个亮度：

```shell
echo 500 > brightness
```

此时会报出Permission denied错误。这是因为我们没有资格修改`/sys`目录下的文件。那么我们自然会想到这样做：

```shell
sudo echo 500 > brightness
```

但是这时候还是会报一模一样的错，为什么？这其实和[[#1.5 pipe|pipe]]中介绍的机制是一样的。cat和grep；ls和tail，它们都只是合作关系，cat不知道grep对它的输出做了什么；同时tail也不知道它的输入是从哪儿来的(只知道是从自己的输入流来的)。**装载输入流和输出流的操作是shell完成的**。在这个例子中，我们只知道，执行sudo这个程序，传递参数echo和500。因此我会以超级用户去执行echo程序，输出500；之后重定向到brightness文件中。但是此时因为重定向的工作是shell去做的，而shell也是当前用户执行的，所以它没有权限修改brightness文件，从而报错。所以我们只能切换到超级用户之后再执行重定向的工作：

```shell
sudo su
echo 500 > brightness
```

同时我们也能发现，终端的开始符号也从`$`变成了`#`。这也是区分root用户和普通用户的方式。

---

另外，除了这种方式，我们还可以借助tee这个程序：

```shell
echo 1000 | sudo tee brightness
```

tee其实就是从输入流拿到序列，写到输出流中。因此它通过pipe从echo的输出流拿到了1000。然后写入到brightness文件中。但是由于这里是sudo执行的，所以不会产生错误。接下来我们看一个例子。`/sys/class`中有一个`leds/`目录，这里存的是电脑上的灯。所以如果我进入`input1::capslock`目录，将其中的brightness文件做如下修改：

```shell
echo 1 | sudo tee brightness
```

这时就会发现电脑的CapsLock键盘灯已经亮了。

# 2. Shell Tools and Scripting

## 2.1 variable

我们可以在命令行中定义变量：

```shell
foo=bar
echo $foo
```



在字符串中也可以嵌套这些变量(这其实才是最主要的用途)：

```shell
echo "value is $foo"
```



但是要注意两点。首先是空格的问题，这个问题[[#1.3 escape|第一章]]也提到了：

```shell
# error, should not use space
foo = bar
```

在这种情况下，shell会把foo当成程序，等号和bar会被当作参数，自然就会报错了：



第二点是单引号和双引号的问题。双引号中的变量可以被替换，而单引号不行：

```shell
echo "value is $foo" # value is bar
echo 'value is $foo' # value is $foo
```

## 2.2 script fuction

我们可以把命令行的命令封装成函数。比如我们可以定义一个创建目录并进入它的函数：

```shell
# mcd.sh
mcd(){
	mkdir -p "$1"
	cd "$1"
}
```

这里的`$1`是什么之后再说，先运行一下这个命令：

```shell
source mcd.sh
mcd test
```

这样就能够创建test目录并进入它。这里我们其实就能看出来，test就是\$1。在脚本语言中，\$符号就是用来表示参数的占位符的。加上一个数字就表示第几个参数，显然，\$0就表示第零个参数，也就是程序本身。接下来我们来一个复杂的例子：

```shell
echo "starting program at"
```

这个例子中的