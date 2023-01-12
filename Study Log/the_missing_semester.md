---
author: "Spread Zhao"
title: the_missing_semester
category: self_study
description: MIT学校的The Missing Section课程，能学到一些非常有用的编程知识和技巧
link: https://missing.csail.mit.edu/2020/
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

## 1.7 Lecture notes

### 1.7.1 Motivation

As computer scientists, we know that computers are great at aiding in repetitive tasks. However, far too often, we forget that this applies just as much to our _use_ of the computer as it does to the computations we want our programs to perform. We have a vast range of tools available at our fingertips that enable us to be more productive and solve more complex problems when working on any computer-related problem. Yet many of us utilize only a small fraction of those tools; we only know enough magical incantations by rote to get by, and blindly copy-paste commands from the internet when we get stuck.

This class is an attempt to address this.

We want to teach you how to make the most of the tools you know, show you new tools to add to your toolbox, and hopefully instill in you some excitement for exploring (and perhaps building) more tools on your own. This is what we believe to be the missing semester from most Computer Science curricula.

### 1.7.2 Class structure

The class consists of 11 1-hour lectures, each one centering on a [particular topic](https://missing.csail.mit.edu/2020/). The lectures are largely independent, though as the semester goes on we will presume that you are familiar with the content from the earlier lectures. We have lecture notes online, but there will be a lot of content covered in class (e.g. in the form of demos) that may not be in the notes. We will be recording lectures and posting the recordings online.

We are trying to cover a lot of ground over the course of just 11 1-hour lectures, so the lectures are fairly dense. To allow you some time to get familiar with the content at your own pace, each lecture includes a set of exercises that guide you through the lecture’s key points. After each lecture, we are hosting office hours where we will be present to help answer any questions you might have. If you are attending the class online, you can send us questions at [missing-semester@mit.edu](mailto:missing-semester@mit.edu).

Due to the limited time we have, we won’t be able to cover all the tools in the same level of detail a full-scale class might. Where possible, we will try to point you towards resources for digging further into a tool or topic, but if something particularly strikes your fancy, don’t hesitate to reach out to us and ask for pointers!

### 1.7.3 The Shell

#### 1.7.3.1 What is the shell?

Computers these days have a variety of interfaces for giving them commands; fanciful graphical user interfaces, voice interfaces, and even AR/VR are everywhere. These are great for 80% of use-cases, but they are often fundamentally restricted in what they allow you to do — you cannot press a button that isn’t there or give a voice command that hasn’t been programmed. To take full advantage of the tools your computer provides, we have to go old-school and drop down to a textual interface: The Shell.

Nearly all platforms you can get your hand on has a shell in one form or another, and many of them have several shells for you to choose from. While they may vary in the details, at their core they are all roughly the same: they allow you to run programs, give them input, and inspect their output in a semi-structured way.

In this lecture, we will focus on the Bourne Again SHell, or “bash” for short. This is one of the most widely used shells, and its syntax is similar to what you will see in many other shells. To open a shell _prompt_ (where you can type commands), you first need a _terminal_. Your device probably shipped with one installed, or you can install one fairly easily.

#### 1.7.3.2 Using the shell

When you launch your terminal, you will see a _prompt_ that often looks a little like this:

```shell
missing:~$ 
```

This is the main textual interface to the shell. It tells you that you are on the machine `missing` and that your “current working directory”, or where you currently are, is `~` (short for “home”). The `$` tells you that you are not the root user (more on that later). At this prompt you can type a _command_, which will then be interpreted by the shell. The most basic command is to execute a program:

```shell
missing:~$ date
Fri 10 Jan 2020 11:49:31 AM EST
missing:~$ 
```

Here, we executed the `date` program, which (perhaps unsurprisingly) prints the current date and time. The shell then asks us for another command to execute. We can also execute a command with _arguments_:

```shell
missing:~$ echo hello
hello
```

In this case, we told the shell to execute the program `echo` with the argument `hello`. The `echo` program simply prints out its arguments. The shell parses the command by splitting it by whitespace, and then runs the program indicated by the first word, supplying each subsequent word as an argument that the program can access. If you want to provide an argument that contains spaces or other special characters (e.g., a directory named “My Photos”), you can either quote the argument with `'` or `"` (`"My Photos"`), or escape just the relevant characters with `\` (`My\ Photos`).

But how does the shell know how to find the `date` or `echo` programs? Well, the shell is a programming environment, just like Python or Ruby, and so it has variables, conditionals, loops, and functions (next lecture!). When you run commands in your shell, you are really writing a small bit of code that your shell interprets. If the shell is asked to execute a command that doesn’t match one of its programming keywords, it consults an _environment variable_ called `$PATH` that lists which directories the shell should search for programs when it is given a command:

```shell
missing:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
missing:~$ which echo
/bin/echo
missing:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

When we run the `echo` command, the shell sees that it should execute the program `echo`, and then searches through the `:`-separated list of directories in `$PATH` for a file by that name. When it finds it, it runs it (assuming the file is _executable_; more on that later). We can find out which file is executed for a given program name using the `which` program. We can also bypass `$PATH` entirely by giving the _path_ to the file we want to execute.

#### 1.7.3.3 Navigating in the shell

A path on the shell is a delimited list of directories; separated by `/` on Linux and macOS and `\` on Windows. On Linux and macOS, the path `/` is the “root” of the file system, under which all directories and files lie, whereas on Windows there is one root for each disk partition (e.g., `C:\`). We will generally assume that you are using a Linux filesystem in this class. A path that starts with `/` is called an _absolute_ path. Any other path is a _relative_ path. Relative paths are relative to the current working directory, which we can see with the `pwd` command and change with the `cd` command. In a path, `.` refers to the current directory, and `..` to its parent directory:

```shell
missing:~$ pwd
/home/missing
missing:~$ cd /home
missing:/home$ pwd
/home
missing:/home$ cd ..
missing:/$ pwd
/
missing:/$ cd ./home
missing:/home$ pwd
/home
missing:/home$ cd missing
missing:~$ pwd
/home/missing
missing:~$ ../../bin/echo hello
hello
```

Notice that our shell prompt kept us informed about what our current working directory was. You can configure your prompt to show you all sorts of useful information, which we will cover in a later lecture.

In general, when we run a program, it will operate in the current directory unless we tell it otherwise. For example, it will usually search for files there, and create new files there if it needs to.

To see what lives in a given directory, we use the `ls` command:

```shell
missing:~$ ls
missing:~$ cd ..
missing:/home$ ls
missing
missing:/home$ cd ..
missing:/$ ls
bin
boot
dev
etc
home
...
```

Unless a directory is given as its first argument, `ls` will print the contents of the current directory. Most commands accept flags and options (flags with values) that start with `-` to modify their behavior. Usually, running a program with the `-h` or `--help` flag will print some help text that tells you what flags and options are available. For example, `ls --help` tells us:

```shell
  -l                         use a long listing format
```

```shell
missing:~$ ls -l /home
drwxr-xr-x 1 missing  users  4096 Jun 15  2019 missing
```

This gives us a bunch more information about each file or directory present. First, the `d` at the beginning of the line tells us that `missing` is a directory. Then follow three groups of three characters (`rwx`). These indicate what permissions the owner of the file (`missing`), the owning group (`users`), and everyone else respectively have on the relevant item. A `-` indicates that the given principal does not have the given permission. Above, only the owner is allowed to modify (`w`) the `missing` directory (i.e., add/remove files in it). To enter a directory, a user must have “search” (represented by “execute”: `x`) permissions on that directory (and its parents). To list its contents, a user must have read (`r`) permissions on that directory. For files, the permissions are as you would expect. Notice that nearly all the files in `/bin` have the `x` permission set for the last group, “everyone else”, so that anyone can execute those programs.

Some other handy programs to know about at this point are `mv` (to rename/move a file), `cp` (to copy a file), and `mkdir` (to make a new directory).

If you ever want _more_ information about a program’s arguments, inputs, outputs, or how it works in general, give the `man` program a try. It takes as an argument the name of a program, and shows you its _manual page_. Press `q` to exit.

```shell
missing:~$ man ls
```

#### 1.7.3.4 Connecting programs

In the shell, programs have two primary “streams” associated with them: their input stream and their output stream. When the program tries to read input, it reads from the input stream, and when it prints something, it prints to its output stream. Normally, a program’s input and output are both your terminal. That is, your keyboard as input and your screen as output. However, we can also rewire those streams!

The simplest form of redirection is `< file` and `> file`. These let you rewire the input and output streams of a program to a file respectively:

```shell
missing:~$ echo hello > hello.txt
missing:~$ cat hello.txt
hello
missing:~$ cat < hello.txt
hello
missing:~$ cat < hello.txt > hello2.txt
missing:~$ cat hello2.txt
hello
```

Demonstrated in the example above, `cat` is a program that con`cat`enates files. When given file names as arguments, it prints the contents of each of the files in sequence to its output stream. But when `cat` is not given any arguments, it prints contents from its input stream to its output stream (like in the third example above).

You can also use `>>` to append to a file. Where this kind of input/output redirection really shines is in the use of _pipes_. The `|` operator lets you “chain” programs such that the output of one is the input of another:

```shell
missing:~$ ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var
missing:~$ curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219
```

We will go into a lot more detail about how to take advantage of pipes in the lecture on data wrangling.

#### 1.7.3.5 A versatile and powerful tool

On most Unix-like systems, one user is special: the “root” user. You may have seen it in the file listings above. The root user is above (almost) all access restrictions, and can create, read, update, and delete any file in the system. You will not usually log into your system as the root user though, since it’s too easy to accidentally break something. Instead, you will be using the `sudo` command. As its name implies, it lets you “do” something “as su” (short for “super user”, or “root”). When you get permission denied errors, it is usually because you need to do something as root. Though make sure you first double-check that you really wanted to do it that way!

One thing you need to be root in order to do is writing to the `sysfs` file system mounted under `/sys`. `sysfs` exposes a number of kernel parameters as files, so that you can easily reconfigure the kernel on the fly without specialized tools. **Note that sysfs does not exist on Windows or macOS.**

For example, the brightness of your laptop’s screen is exposed through a file called `brightness` under

```shell
/sys/class/backlight
```

By writing a value into that file, we can change the screen brightness. Your first instinct might be to do something like:

```shell
$ sudo find -L /sys/class/backlight -maxdepth 2 -name '*brightness*'
/sys/class/backlight/thinkpad_screen/brightness
$ cd /sys/class/backlight/thinkpad_screen
$ sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied
```

This error may come as a surprise. After all, we ran the command with `sudo`! This is an important thing to know about the shell. Operations like `|`, `>`, and `<` are done _by the shell_, not by the individual program. `echo` and friends do not “know” about `|`. They just read from their input and write to their output, whatever it may be. In the case above, the _shell_ (which is authenticated just as your user) tries to open the brightness file for writing, before setting that as `sudo echo`’s output, but is prevented from doing so since the shell does not run as root. Using this knowledge, we can work around this:

```shell
$ echo 3 | sudo tee brightness
```

Since the `tee` program is the one to open the `/sys` file for writing, and _it_ is running as `root`, the permissions all work out. You can control all sorts of fun and useful things through `/sys`, such as the state of various system LEDs (your path might be different):

```shell
$ echo 1 | sudo tee /sys/class/leds/input6::scrolllock/brightness
```

### 1.7.4 Next steps

At this point you know your way around a shell enough to accomplish basic tasks. You should be able to navigate around to find files of interest and use the basic functionality of most programs. In the next lecture, we will talk about how to perform and automate more complex tasks using the shell and the many handy command-line programs out there.

### 1.7.5 Exercises

All classes in this course are accompanied by a series of exercises. Some give you a specific task to do, while others are open-ended, like “try using X and Y programs”. We highly encourage you to try them out.

We have not written solutions for the exercises. If you are stuck on anything in particular, feel free to send us an email describing what you’ve tried so far, and we will try to help you out.

1.  For this course, you need to be using a Unix shell like Bash or ZSH. If you are on Linux or macOS, you don’t have to do anything special. If you are on Windows, you need to make sure you are not running cmd.exe or PowerShell; you can use [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) or a Linux virtual machine to use Unix-style command-line tools. To make sure you’re running an appropriate shell, you can try the command `echo $SHELL`. If it says something like `/bin/bash` or `/usr/bin/zsh`, that means you’re running the right program.
2.  Create a new directory called `missing` under `/tmp`.
3.  Look up the `touch` program. The `man` program is your friend.
4.  Use `touch` to create a new file called `semester` in `missing`.
5.  Write the following into that file, one line at a time:
    
    ```shell
    #!/bin/sh
    curl --head --silent https://missing.csail.mit.edu
    ```
    
    The first line might be tricky to get working. It’s helpful to know that `#` starts a comment in Bash, and `!` has a special meaning even within double-quoted (`"`) strings. Bash treats single-quoted strings (`'`) differently: they will do the trick in this case. See the Bash [quoting](https://www.gnu.org/software/bash/manual/html_node/Quoting.html) manual page for more information.
    
6.  Try to execute the file, i.e. type the path to the script (`./semester`) into your shell and press enter. Understand why it doesn’t work by consulting the output of `ls` (hint: look at the permission bits of the file).
7.  Run the command by explicitly starting the `sh` interpreter, and giving it the file `semester` as the first argument, i.e. `sh semester`. Why does this work, while `./semester` didn’t?
8.  Look up the `chmod` program (e.g. use `man chmod`).
9.  Use `chmod` to make it possible to run the command `./semester` rather than having to type `sh semester`. How does your shell know that the file is supposed to be interpreted using `sh`? See this page on the [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) line for more information.
10.  Use `|` and `>` to write the “last modified” date output by `semester` into a file called `last-modified.txt` in your home directory.
11.  Write a command that reads out your laptop battery’s power level or your desktop machine’s CPU temperature from `/sys`. Note: if you’re a macOS user, your OS doesn’t have sysfs, so you can skip this exercise.

# 2. Shell Tools and Scripting

## 2.1 variable

我们可以在命令行中定义变量：

```shell
foo=bar
echo $foo
```

![[Study Log/resources/Pasted image 20221216144012.png]]

在字符串中也可以嵌套这些变量(这其实才是最主要的用途)：

```shell
echo "value is $foo"
```

![[Study Log/resources/Pasted image 20221216144030.png]]

但是要注意两点。首先是空格的问题，这个问题[[#1.3 escape|第一章]]也提到了：

```shell
# error, should not use space
foo = bar
```

在这种情况下，shell会把foo当成程序，等号和bar会被当作参数，自然就会报错了：

![[Study Log/resources/Pasted image 20221216144054.png]]

第二点是单引号和双引号的问题。双引号中的变量可以被替换，而单引号不行：

```shell
echo "value is $foo" # value is bar
echo 'value is $foo' # value is $foo
```

![[Study Log/resources/Pasted image 20221216144114.png]]

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
#!/usr/bin/bash
echo "Starting program at $(date)"

echo "running program $0 with $# arguments with pid $$"

for file in "$@"; do
	grep foobar "$file" > /dev/null 2> /dev/null
	if [[ "$?" -ne 0 ]]; then
		echo "file $file does not have any foobar, adding one"
		echo "# foobar" >> "$file"
	fi
done
```

这个例子中，我们首先输出当前的时间，使用`$(...)`来保存程序执行的结果，并将其嵌套入字符串中。接下来，就是\$0，也就是程序的名字。\$\#表示参数的个数；\$\$表示该进程的pid。然后是一个for循环，在其中的每个变量file都是一个字符串，比如这样来执行：

```shell
sudo ./example.sh bashtest1 bashtest2 bashtest3
```

那么file就会从bashtest1一直遍历到bashtest3。而\$@符号表示所有的参数列表。在for循环内部，我们进行了grep操作。从"\$file"这个字符串中找有没有foobar这段字符。如果有，那么我们将其重定向到`/dev/null`中。**这里注意后面的2>**，这表示如果出现错误，比如没有foobar这段字符，那么就用**标准错误流**将其输出到/dev/null中。

然后，我们判断`$?`和0是否相等。`$?`会返回上个命令的错误码，如果是0则表示成功。因此如果不想等(-ne)的话，就手动添加一个。

以上的脚本文件执行起来就是这样的。如果我们有测试文件bashtest1, bashtest2, bashtest3。其中的内容是这样的：

![[Study Log/resources/Pasted image 20221216150511.png]]

那么这个脚本的执行结果就是：

![[Study Log/resources/Pasted image 20221216150651.png]]

> 能看到，由于1和3中都没有foobar，所以它在后面添加了一个。

## 2.3 wildcard

在脚本语言中也可以使用各种通配符。比如我有如下文件：

![[Study Log/resources/Pasted image 20221216150900.png|200]]

那么注意观察下面两条命令的执行结果，就明白是什么意思了。

![[Study Log/resources/Pasted image 20221216151047.png]]

单个问号只占一个位置，而星号表示任意位数的字符串。

---

如果我想一次创建多个文件夹，可以这样：

![[Study Log/resources/Pasted image 20221216151229.png]]

这其实就相当于下面的命令：

```shell
mkdir test1 test2 test3
```

甚至还可以进行循环和嵌套：

![[Study Log/resources/Pasted image 20221216151428.png]]

## 2.4 process substitution

从2.1的介绍我们能知道，如果想查看某些特殊的输出，比如想同时输出当前目录的文件和父目录的文件，可以这样：

![[Study Log/resources/Pasted image 20221216152200.png]]

> 这里的-e表示翻译转义字符，比如里面的\n。

但是还有另外一种方式，就是process substitution：

![[Study Log/resources/Pasted image 20221216152317.png]]

`<(...)`符号的工作原理是，让程序执行的结果放在一个临时文件中。如何证明呢？用一个程序：diff。它比较的是文件，能输出它们的不同。下面的例子很好看懂：

![[Study Log/resources/Pasted image 20221216152800.png]]

> 这里将ls程序执行的结果放到了临时文件中，并比较这两个临时文件的不同。

## 2.5 logic cal

在脚本语言里也可以写逻辑运算，只不过和程序语言中有一些不同。比如下面的例子：

```shell
false || echo haha # will run
true || echo hehe # will not run
false && echo haha # will not run
true && echo hehe # will run
```

这些程序都是从左到右执行的。或运算是：左边执行失败了，就执行右边，如果左边成功了，就不执行右边了；而与运算是：只要有一个执行失败就结束。所以这些程序的输出是这样的：

![[Study Log/resources/Pasted image 20221216153320.png]]

## 2.6 nifty tools

下面介绍一些实用的小工具。首先是tldr，它相当于一个精简的man，只列出有用的帮助：

![[Study Log/resources/Pasted image 20221216153629.png]]

---

find，这个很常用，就是搜索。下面给出一个比较常用的例子：

```shell
find . -name "*.sh" -type f # 找文件
find . -name "test*" -type d # 找文件夹
```

![[Study Log/resources/Pasted image 20221216153921.png]]

---

locate，这个就是有索引地找。

![[Study Log/resources/Pasted image 20221216154025.png]]

它比find快很多，但是需要建立索引的过程：

```shell
sudo updatedb
```

## 2.7 Lecture notes

In this lecture, we will present some of the basics of using bash as a scripting language along with a number of shell tools that cover several of the most common tasks that you will be constantly performing in the command line.

### 2.7.1 Shell Scripting

So far we have seen how to execute commands in the shell and pipe them together. However, in many scenarios you will want to perform a series of commands and make use of control flow expressions like conditionals or loops.

Shell scripts are the next step in complexity. Most shells have their own scripting language with variables, control flow and its own syntax. What makes shell scripting different from other scripting programming language is that it is optimized for performing shell-related tasks. Thus, creating command pipelines, saving results into files, and reading from standard input are primitives in shell scripting, which makes it easier to use than general purpose scripting languages. For this section we will focus on bash scripting since it is the most common.

To assign variables in bash, use the syntax `foo=bar` and access the value of the variable with `$foo`. Note that `foo = bar` will not work since it is interpreted as calling the `foo` program with arguments `=` and `bar`. In general, in shell scripts the space character will perform argument splitting. This behavior can be confusing to use at first, so always check for that.

Strings in bash can be defined with `'` and `"` delimiters, but they are not equivalent. Strings delimited with `'` are literal strings and will not substitute variable values whereas `"` delimited strings will.

```shell
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo
```

As with most programming languages, bash supports control flow techniques including `if`, `case`, `while` and `for`. Similarly, `bash` has functions that take arguments and can operate with them. Here is an example of a function that creates a directory and `cd`s into it.

```shell
mcd () {
    mkdir -p "$1"
    cd "$1"
}
```

Here `$1` is the first argument to the script/function. Unlike other scripting languages, bash uses a variety of special variables to refer to arguments, error codes, and other relevant variables. Below is a list of some of them. A more comprehensive list can be found [here](https://tldp.org/LDP/abs/html/special-chars.html).

-   `$0` - Name of the script
-   `$1` to `$9` - Arguments to the script. `$1` is the first argument and so on.
-   `$@` - All the arguments
-   `$#` - Number of arguments
-   `$?` - Return code of the previous command
-   `$$` - Process identification number (PID) for the current script
-   `!!` - Entire last command, including arguments. A common pattern is to execute a command only for it to fail due to missing permissions; you can quickly re-execute the command with sudo by doing `sudo !!`
-   `$_` - Last argument from the last command. If you are in an interactive shell, you can also quickly get this value by typing `Esc` followed by `.` or `Alt+.`

Commands will often return output using `STDOUT`, errors through `STDERR`, and a Return Code to report errors in a more script-friendly manner. The return code or exit status is the way scripts/commands have to communicate how execution went. A value of 0 usually means everything went OK; anything different from 0 means an error occurred.

Exit codes can be used to conditionally execute commands using `&&` (and operator) and `||` (or operator), both of which are [short-circuiting](https://en.wikipedia.org/wiki/Short-circuit_evaluation) operators. Commands can also be separated within the same line using a semicolon `;`. The `true` program will always have a 0 return code and the `false` command will always have a 1 return code. Let’s see some examples

```shell
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
#

true ; echo "This will always run"
# This will always run

false ; echo "This will always run"
# This will always run
```

Another common pattern is wanting to get the output of a command as a variable. This can be done with _command substitution_. Whenever you place `$( CMD )` it will execute `CMD`, get the output of the command and substitute it in place. For example, if you do `for file in $(ls)`, the shell will first call `ls` and then iterate over those values. A lesser known similar feature is _process substitution_, `<( CMD )` will execute `CMD` and place the output in a temporary file and substitute the `<()` with that file’s name. This is useful when commands expect values to be passed by file instead of by STDIN. For example, `diff <(ls foo) <(ls bar)` will show differences between files in dirs `foo` and `bar`.

Since that was a huge information dump, let’s see an example that showcases some of these features. It will iterate through the arguments we provide, `grep` for the string `foobar`, and append it to the file as a comment if it’s not found.

```shell
#!/bin/bash

echo "Starting program at $(date)" # Date will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do
    grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```

In the comparison we tested whether `$?` was not equal to 0. Bash implements many comparisons of this sort - you can find a detailed list in the manpage for [`test`](https://www.man7.org/linux/man-pages/man1/test.1.html). When performing comparisons in bash, try to use double brackets `[[ ]]` in favor of simple brackets `[ ]`. Chances of making mistakes are lower although it won’t be portable to `sh`. A more detailed explanation can be found [here](http://mywiki.wooledge.org/BashFAQ/031).

When launching scripts, you will often want to provide arguments that are similar. Bash has ways of making this easier, expanding expressions by carrying out filename expansion. These techniques are often referred to as shell _globbing_.

-   Wildcards - Whenever you want to perform some sort of wildcard matching, you can use `?` and `*` to match one or any amount of characters respectively. For instance, given files `foo`, `foo1`, `foo2`, `foo10` and `bar`, the command `rm foo?` will delete `foo1` and `foo2` whereas `rm foo*` will delete all but `bar`.
-   Curly braces `{}` - Whenever you have a common substring in a series of commands, you can use curly braces for bash to expand this automatically. This comes in very handy when moving or converting files.

```shell
convert image.{png,jpg}
# Will expand to
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# Will expand to
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder
# Will move all *.py and *.sh files


mkdir foo bar
# This creates files foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h
touch {foo,bar}/{a..h}
touch foo/x bar/y
# Show differences between files in foo and bar
diff <(ls foo) <(ls bar)
# Outputs
# < x
# ---
# > y
```

Writing `bash` scripts can be tricky and unintuitive. There are tools like [shellcheck](https://github.com/koalaman/shellcheck) that will help you find errors in your sh/bash scripts.

Note that scripts need not necessarily be written in bash to be called from the terminal. For instance, here’s a simple Python script that outputs its arguments in reversed order:

```shell
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

The kernel knows to execute this script with a python interpreter instead of a shell command because we included a [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) line at the top of the script. It is good practice to write shebang lines using the [`env`](https://www.man7.org/linux/man-pages/man1/env.1.html) command that will resolve to wherever the command lives in the system, increasing the portability of your scripts. To resolve the location, `env` will make use of the `PATH` environment variable we introduced in the first lecture. For this example the shebang line would look like `#!/usr/bin/env python`.

Some differences between shell functions and scripts that you should keep in mind are:

-   Functions have to be in the same language as the shell, while scripts can be written in any language. This is why including a shebang for scripts is important.
-   Functions are loaded once when their definition is read. Scripts are loaded every time they are executed. This makes functions slightly faster to load, but whenever you change them you will have to reload their definition.
-   Functions are executed in the current shell environment whereas scripts execute in their own process. Thus, functions can modify environment variables, e.g. change your current directory, whereas scripts can’t. Scripts will be passed by value environment variables that have been exported using [`export`](https://www.man7.org/linux/man-pages/man1/export.1p.html)
-   As with any programming language, functions are a powerful construct to achieve modularity, code reuse, and clarity of shell code. Often shell scripts will include their own function definitions.

### 2.7.2 Shell Tools

#### 2.7.2.1 Finding how to use commands

At this point, you might be wondering how to find the flags for the commands in the aliasing section such as `ls -l`, `mv -i` and `mkdir -p`. More generally, given a command, how do you go about finding out what it does and its different options? You could always start googling, but since UNIX predates StackOverflow, there are built-in ways of getting this information.

As we saw in the shell lecture, the first-order approach is to call said command with the `-h` or `--help` flags. A more detailed approach is to use the `man` command. Short for manual, [`man`](https://www.man7.org/linux/man-pages/man1/man.1.html) provides a manual page (called manpage) for a command you specify. For example, `man rm` will output the behavior of the `rm` command along with the flags that it takes, including the `-i` flag we showed earlier. In fact, what I have been linking so far for every command is the online version of the Linux manpages for the commands. Even non-native commands that you install will have manpage entries if the developer wrote them and included them as part of the installation process. For interactive tools such as the ones based on ncurses, help for the commands can often be accessed within the program using the `:help` command or typing `?`.

Sometimes manpages can provide overly detailed descriptions of the commands, making it hard to decipher what flags/syntax to use for common use cases. [TLDR pages](https://tldr.sh/) are a nifty complementary solution that focuses on giving example use cases of a command so you can quickly figure out which options to use. For instance, I find myself referring back to the tldr pages for [`tar`](https://tldr.ostera.io/tar) and [`ffmpeg`](https://tldr.ostera.io/ffmpeg) way more often than the manpages.

#### 2.7.2.2 Finding files

One of the most common repetitive tasks that every programmer faces is finding files or directories. All UNIX-like systems come packaged with [`find`](https://www.man7.org/linux/man-pages/man1/find.1.html), a great shell tool to find files. `find` will recursively search for files matching some criteria. Some examples:

```shell
# Find all directories named src
find . -name src -type d
# Find all python files that have a folder named test in their path
find . -path '*/test/*.py' -type f
# Find all files modified in the last day
find . -mtime -1
# Find all zip files with size in range 500k to 10M
find . -size +500k -size -10M -name '*.tar.gz'
```

Beyond listing files, find can also perform actions over files that match your query. This property can be incredibly helpful to simplify what could be fairly monotonous tasks.

```shell
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \;
# Find all PNG files and convert them to JPG
find . -name '*.png' -exec convert {} {}.jpg \;
```

Despite `find`’s ubiquitousness, its syntax can sometimes be tricky to remember. For instance, to simply find files that match some pattern `PATTERN` you have to execute `find -name '*PATTERN*'` (or `-iname` if you want the pattern matching to be case insensitive). You could start building aliases for those scenarios, but part of the shell philosophy is that it is good to explore alternatives. Remember, one of the best properties of the shell is that you are just calling programs, so you can find (or even write yourself) replacements for some. For instance, [`fd`](https://github.com/sharkdp/fd) is a simple, fast, and user-friendly alternative to `find`. It offers some nice defaults like colorized output, default regex matching, and Unicode support. It also has, in my opinion, a more intuitive syntax. For example, the syntax to find a pattern `PATTERN` is `fd PATTERN`.

Most would agree that `find` and `fd` are good, but some of you might be wondering about the efficiency of looking for files every time versus compiling some sort of index or database for quickly searching. That is what [`locate`](https://www.man7.org/linux/man-pages/man1/locate.1.html) is for. `locate` uses a database that is updated using [`updatedb`](https://www.man7.org/linux/man-pages/man1/updatedb.1.html). In most systems, `updatedb` is updated daily via [`cron`](https://www.man7.org/linux/man-pages/man8/cron.8.html). Therefore one trade-off between the two is speed vs freshness. Moreover `find` and similar tools can also find files using attributes such as file size, modification time, or file permissions, while `locate` just uses the file name. A more in-depth comparison can be found [here](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other).

#### 2.7.2.3 Finding code

Finding files by name is useful, but quite often you want to search based on file _content_. A common scenario is wanting to search for all files that contain some pattern, along with where in those files said pattern occurs. To achieve this, most UNIX-like systems provide [`grep`](https://www.man7.org/linux/man-pages/man1/grep.1.html), a generic tool for matching patterns from the input text. `grep` is an incredibly valuable shell tool that we will cover in greater detail during the data wrangling lecture.

For now, know that `grep` has many flags that make it a very versatile tool. Some I frequently use are `-C` for getting **C**ontext around the matching line and `-v` for in**v**erting the match, i.e. print all lines that do **not** match the pattern. For example, `grep -C 5` will print 5 lines before and after the match. When it comes to quickly searching through many files, you want to use `-R` since it will **R**ecursively go into directories and look for files for the matching string.

But `grep -R` can be improved in many ways, such as ignoring `.git` folders, using multi CPU support, &c. Many `grep` alternatives have been developed, including [ack](https://github.com/beyondgrep/ack3), [ag](https://github.com/ggreer/the_silver_searcher) and [rg](https://github.com/BurntSushi/ripgrep). All of them are fantastic and pretty much provide the same functionality. For now I am sticking with ripgrep (`rg`), given how fast and intuitive it is. Some examples:

```shell
# Find all python files where I used the requests library
rg -t py 'import requests'
# Find all files (including hidden files) without a shebang line
rg -u --files-without-match "^#!"
# Find all matches of foo and print the following 5 lines
rg foo -A 5
# Print statistics of matches (# of matched lines and files )
rg --stats PATTERN
```

Note that as with `find`/`fd`, it is important that you know that these problems can be quickly solved using one of these tools, while the specific tools you use are not as important.

#### 2.7.2.4 Finding shell commands

So far we have seen how to find files and code, but as you start spending more time in the shell, you may want to find specific commands you typed at some point. The first thing to know is that typing the up arrow will give you back your last command, and if you keep pressing it you will slowly go through your shell history.

The `history` command will let you access your shell history programmatically. It will print your shell history to the standard output. If we want to search there we can pipe that output to `grep` and search for patterns. `history | grep find` will print commands that contain the substring “find”.

In most shells, you can make use of `Ctrl+R` to perform backwards search through your history. After pressing `Ctrl+R`, you can type a substring you want to match for commands in your history. As you keep pressing it, you will cycle through the matches in your history. This can also be enabled with the UP/DOWN arrows in [zsh](https://github.com/zsh-users/zsh-history-substring-search). A nice addition on top of `Ctrl+R` comes with using [fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r) bindings. `fzf` is a general-purpose fuzzy finder that can be used with many commands. Here it is used to fuzzily match through your history and present results in a convenient and visually pleasing manner.

Another cool history-related trick I really enjoy is **history-based autosuggestions**. First introduced by the [fish](https://fishshell.com/) shell, this feature dynamically autocompletes your current shell command with the most recent command that you typed that shares a common prefix with it. It can be enabled in [zsh](https://github.com/zsh-users/zsh-autosuggestions) and it is a great quality of life trick for your shell.

You can modify your shell’s history behavior, like preventing commands with a leading space from being included. This comes in handy when you are typing commands with passwords or other bits of sensitive information. To do this, add `HISTCONTROL=ignorespace` to your `.bashrc` or `setopt HIST_IGNORE_SPACE` to your `.zshrc`. If you make the mistake of not adding the leading space, you can always manually remove the entry by editing your `.bash_history` or `.zhistory`.

#### 2.7.2.5 Directory Navigation

So far, we have assumed that you are already where you need to be to perform these actions. But how do you go about quickly navigating directories? There are many simple ways that you could do this, such as writing shell aliases or creating symlinks with [ln -s](https://www.man7.org/linux/man-pages/man1/ln.1.html), but the truth is that developers have figured out quite clever and sophisticated solutions by now.

As with the theme of this course, you often want to optimize for the common case. Finding frequent and/or recent files and directories can be done through tools like [`fasd`](https://github.com/clvv/fasd) and [`autojump`](https://github.com/wting/autojump). Fasd ranks files and directories by [_frecency_](https://web.archive.org/web/20210421120120/https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm), that is, by both _frequency_ and _recency_. By default, `fasd` adds a `z` command that you can use to quickly `cd` using a substring of a _frecent_ directory. For example, if you often go to `/home/user/files/cool_project` you can simply use `z cool` to jump there. Using autojump, this same change of directory could be accomplished using `j cool`.

More complex tools exist to quickly get an overview of a directory structure: [`tree`](https://linux.die.net/man/1/tree), [`broot`](https://github.com/Canop/broot) or even full fledged file managers like [`nnn`](https://github.com/jarun/nnn) or [`ranger`](https://github.com/ranger/ranger).

### 2.7.3 Exercises

1.  Read [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) and write an `ls` command that lists files in the following manner
    
    -   Includes all files, including hidden files
    -   Sizes are listed in human readable format (e.g. 454M instead of 454279954)
    -   Files are ordered by recency
    -   Output is colorized
    
    A sample output would look like this
    
    ```
     -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
     drwxr-xr-x   5 user group  160 Jan 14 09:53 .
     -rw-r--r--   1 user group  514 Jan 14 06:42 bar
     -rw-r--r--   1 user group 106M Jan 13 12:12 foo
     drwx------+ 47 user group 1.5K Jan 12 18:08 ..
    ```
    
2.  Write bash functions `marco` and `polo` that do the following. Whenever you execute `marco` the current working directory should be saved in some manner, then when you execute `polo`, no matter what directory you are in, `polo` should `cd` you back to the directory where you executed `marco`. For ease of debugging you can write the code in a file `marco.sh` and (re)load the definitions to your shell by executing `source marco.sh`.
    
3.  Say you have a command that fails rarely. In order to debug it you need to capture its output but it can be time consuming to get a failure run. Write a bash script that runs the following script until it fails and captures its standard output and error streams to files and prints everything at the end. Bonus points if you can also report how many runs it took for the script to fail.
    
    ```
     #!/usr/bin/env bash
    
     n=$(( RANDOM % 100 ))
    
     if [[ n -eq 42 ]]; then
        echo "Something went wrong"
        >&2 echo "The error was using magic numbers"
        exit 1
     fi
    
     echo "Everything went according to plan"
    ```
    
4.  As we covered in the lecture `find`’s `-exec` can be very powerful for performing operations over the files we are searching for. However, what if we want to do something with **all** the files, like creating a zip file? As you have seen so far commands will take input from both arguments and STDIN. When piping commands, we are connecting STDOUT to STDIN, but some commands like `tar` take inputs from arguments. To bridge this disconnect there’s the [`xargs`](https://www.man7.org/linux/man-pages/man1/xargs.1.html) command which will execute a command using STDIN as arguments. For example `ls | xargs rm` will delete the files in the current directory.
    
    Your task is to write a command that recursively finds all HTML files in the folder and makes a zip with them. Note that your command should work even if the files have spaces (hint: check `-d` flag for `xargs`).
    
    If you’re on macOS, note that the default BSD `find` is different from the one included in [GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands). You can use `-print0` on `find` and the `-0` flag on `xargs`. As a macOS user, you should be aware that command-line utilities shipped with macOS may differ from the GNU counterparts; you can install the GNU versions if you like by [using brew](https://formulae.brew.sh/formula/coreutils).
    
5.  (Advanced) Write a command or script to recursively find the most recently modified file in a directory. More generally, can you list all files by recency?

# 3. Vim

vim这种东西还是得多用才行。所以我直接贴了官方的学习笔记，列出一些重点。

Writing English words and writing code are very different activities. When programming, you spend more time switching files, reading, navigating, and editing code compared to writing a long stream. It makes sense that there are different types of programs for writing English words versus code (e.g. Microsoft Word versus Visual Studio Code).

As programmers, we spend most of our time editing code, so it’s worth investing time mastering an editor that fits your needs. Here’s how you learn a new editor:

-   Start with a tutorial (i.e. this lecture, plus resources that we point out)
-   Stick with using the editor for all your text editing needs (even if it slows you down initially)
-   Look things up as you go: if it seems like there should be a better way to do something, there probably is

If you follow the above method, fully committing to using the new program for all text editing purposes, the timeline for learning a sophisticated text editor looks like this. In an hour or two, you’ll learn basic editor functions such as opening and editing files, save/quit, and navigating buffers. Once you’re 20 hours in, you should be as fast as you were with your old editor. After that, the benefits start: you will have enough knowledge and muscle memory that using the new editor saves you time. Modern text editors are fancy and powerful tools, so the learning never stops: you’ll get even faster as you learn more.

## 3.1  Which editor to learn?

Programmers have [strong opinions](https://en.wikipedia.org/wiki/Editor_war) about their text editors.

Which editors are popular today? See this [Stack Overflow survey](https://insights.stackoverflow.com/survey/2019/#development-environments-and-tools) (there may be some bias because Stack Overflow users may not be representative of programmers as a whole). [Visual Studio Code](https://code.visualstudio.com/) is the most popular editor. [Vim](https://www.vim.org/) is the most popular command-line-based editor.

### Vim

All the instructors of this class use Vim as their editor. Vim has a rich history; it originated from the Vi editor (1976), and it’s still being developed today. Vim has some really neat ideas behind it, and for this reason, lots of tools support a Vim emulation mode (for example, 1.4 million people have installed [Vim emulation for VS code](https://github.com/VSCodeVim/Vim)). Vim is probably worth learning even if you finally end up switching to some other text editor.

It’s not possible to teach all of Vim’s functionality in 50 minutes, so we’re going to focus on explaining the philosophy of Vim, teaching you the basics, showing you some of the more advanced functionality, and giving you the resources to master the tool.

## 3.2 Philosophy of Vim

When programming, you spend most of your time reading/editing, not writing. For this reason, Vim is a _modal_ editor: it has different modes for inserting text vs manipulating text. Vim is programmable (with Vimscript and also other languages like Python), and Vim’s interface itself is a programming language: keystrokes (with mnemonic names) are commands, and these commands are composable. Vim avoids the use of the mouse, because it’s too slow; Vim even avoids using the arrow keys because it requires too much movement.

The end result is an editor that can match the speed at which you think.

## 3.3 Modal editing

Vim’s design is based on the idea that a lot of programmer time is spent reading, navigating, and making small edits, as opposed to writing long streams of text. For this reason, Vim has multiple operating modes.

-   **Normal**: for moving around a file and making edits
-   **Insert**: for inserting text
-   **Replace**: for replacing text
-   **Visual** (plain, line, or block): for selecting blocks of text
-   **Command-line**: for running a command

Keystrokes have different meanings in different operating modes. For example, the letter `x` in Insert mode will just insert a literal character ‘x’, but in Normal mode, it will delete the character under the cursor, and in Visual mode, it will delete the selection.

In its default configuration, Vim shows the current mode in the bottom left. The initial/default mode is Normal mode. You’ll generally spend most of your time between Normal mode and Insert mode.

You change modes by pressing `<ESC>` (the escape key) to switch from any mode back to Normal mode. From Normal mode, enter Insert mode with `i`, Replace mode with `R`, Visual mode with `v`, Visual Line mode with `V`, Visual Block mode with `<C-v>` (Ctrl-V, sometimes also written `^V`), and Command-line mode with `:`.

You use the `<ESC>` key a lot when using Vim: consider remapping Caps Lock to Escape ([macOS instructions](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS)).

## 3.4 Basics

### Inserting text

From Normal mode, press `i` to enter Insert mode. Now, Vim behaves like any other text editor, until you press `<ESC>` to return to Normal mode. This, along with the basics explained above, are all you need to start editing files using Vim (though not particularly efficiently, if you’re spending all your time editing from Insert mode).

### Buffers, tabs, and windows

Vim maintains a set of open files, called “buffers”. A Vim session has a number of tabs, each of which has a number of windows (split panes). Each window shows a single buffer. Unlike other programs you are familiar with, like web browsers, there is not a 1-to-1 correspondence between buffers and windows; windows are merely views. A given buffer may be open in _multiple_ windows, even within the same tab. This can be quite handy, for example, to view two different parts of a file at the same time.

By default, Vim opens with a single tab, which contains a single window.

### Command-line

Command mode can be entered by typing `:` in Normal mode. Your cursor will jump to the command line at the bottom of the screen upon pressing `:`. This mode has many functionalities, including opening, saving, and closing files, and [quitting Vim](https://twitter.com/iamdevloper/status/435555976687923200).

-   `:q` quit (close window)
-   `:w` save (“write”)
-   `:wq` save and quit
-   `:e {name of file}` open file for editing
-   `:ls` show open buffers
-   `:help {topic}` open help
    -   `:help :w` opens help for the `:w` command
    -   `:help w` opens help for the `w` movement

## 3.5 Vim’s interface is a programming language

The most important idea in Vim is that Vim’s interface itself is a programming language. Keystrokes (with mnemonic names) are commands, and these commands _compose_. This enables efficient movement and edits, especially once the commands become muscle memory.

### Movement

You should spend most of your time in Normal mode, using movement commands to navigate the buffer. Movements in Vim are also called “nouns”, because they refer to chunks of text.

-   Basic movement: `hjkl` (left, down, up, right)
-   Words: `w` (next word), `b` (beginning of word), `e` (end of word)
-   Lines: `0` (beginning of line), `^` (first non-blank character), `$` (end of line)
-   Screen: `H` (top of screen), `M` (middle of screen), `L` (bottom of screen)
-   Scroll: `Ctrl-u` (up), `Ctrl-d` (down)
-   File: `gg` (beginning of file), `G` (end of file)
-   Line numbers: `:{number}<CR>` or `{number}G` (line {number})
-   Misc: `%` (corresponding item)
-   Find: `f{character}`, `t{character}`, `F{character}`, `T{character}`
    -   find/to forward/backward {character} on the current line
    -   `,` / `;` for navigating matches
-   Search: `/{regex}`, `n` / `N` for navigating matches

### Selection

Visual modes:

-   Visual: `v`
-   Visual Line: `V`
-   Visual Block: `Ctrl-v`

Can use movement keys to make selection.

### Edits

Everything that you used to do with the mouse, you now do with the keyboard using editing commands that compose with movement commands. Here’s where Vim’s interface starts to look like a programming language. Vim’s editing commands are also called “verbs”, because verbs act on nouns.

-   `i` enter Insert mode
    -   but for manipulating/deleting text, want to use something more than backspace
-   `o` / `O` insert line below / above
-   `d{motion}` delete {motion}
    -   e.g. `dw` is delete word, `d$` is delete to end of line, `d0` is delete to beginning of line
-   `c{motion}` change {motion}
    -   e.g. `cw` is change word
    -   like `d{motion}` followed by `i`
-   `x` delete character (equal do `dl`)
-   `s` substitute character (equal to `cl`)
-   Visual mode + manipulation
    -   select text, `d` to delete it or `c` to change it
-   `u` to undo, `<C-r>` to redo
-   `y` to copy / “yank” (some other commands like `d` also copy)
-   `p` to paste
-   Lots more to learn: e.g. `~` flips the case of a character

### Counts

You can combine nouns and verbs with a count, which will perform a given action a number of times.

-   `3w` move 3 words forward
-   `5j` move 5 lines down
-   `7dw` delete 7 words

### Modifiers

You can use modifiers to change the meaning of a noun. Some modifiers are `i`, which means “inner” or “inside”, and `a`, which means “around”.

-   `ci(` change the contents inside the current pair of parentheses
-   `ci[` change the contents inside the current pair of square brackets
-   `da'` delete a single-quoted string, including the surrounding single quotes

## 3.6 Demo

Here is a broken [fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz) implementation:

```python
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print('fizz')
        if i % 5 == 0:
            print('fizz')
        if i % 3 and i % 5:
            print(i)

def main():
    fizz_buzz(10)
```

We will fix the following issues:

-   Main is never called
-   Starts at 0 instead of 1
-   Prints “fizz” and “buzz” on separate lines for multiples of 15
-   Prints “fizz” for multiples of 5
-   Uses a hard-coded argument of 10 instead of taking a command-line argument

See the lecture video for the demonstration. Compare how the above changes are made using Vim to how you might make the same edits using another program. Notice how very few keystrokes are required in Vim, allowing you to edit at the speed you think.

## 3.7 Customizing Vim

Vim is customized through a plain-text configuration file in `~/.vimrc` (containing Vimscript commands). There are probably lots of basic settings that you want to turn on.

We are providing a well-documented basic config that you can use as a starting point. We recommend using this because it fixes some of Vim’s quirky default behavior. **Download our config [here](https://missing.csail.mit.edu/2020/files/vimrc) and save it to `~/.vimrc`.**

Vim is heavily customizable, and it’s worth spending time exploring customization options. You can look at people’s dotfiles on GitHub for inspiration, for example, your instructors’ Vim configs ([Anish](https://github.com/anishathalye/dotfiles/blob/master/vimrc), [Jon](https://github.com/jonhoo/configs/blob/master/editor/.config/nvim/init.vim) (uses [neovim](https://neovim.io/)), [Jose](https://github.com/JJGO/dotfiles/blob/master/vim/.vimrc)). There are lots of good blog posts on this topic too. Try not to copy-and-paste people’s full configuration, but read it, understand it, and take what you need.

## 3.8 Extending Vim

There are tons of plugins for extending Vim. Contrary to outdated advice that you might find on the internet, you do _not_ need to use a plugin manager for Vim (since Vim 8.0). Instead, you can use the built-in package management system. Simply create the directory `~/.vim/pack/vendor/start/`, and put plugins in there (e.g. via `git clone`).

Here are some of our favorite plugins:

-   [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim): fuzzy file finder
-   [ack.vim](https://github.com/mileszs/ack.vim): code search
-   [nerdtree](https://github.com/scrooloose/nerdtree): file explorer
-   [vim-easymotion](https://github.com/easymotion/vim-easymotion): magic motions

We’re trying to avoid giving an overwhelmingly long list of plugins here. You can check out the instructors’ dotfiles ([Anish](https://github.com/anishathalye/dotfiles), [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/JJGO/dotfiles)) to see what other plugins we use. Check out [Vim Awesome](https://vimawesome.com/) for more awesome Vim plugins. There are also tons of blog posts on this topic: just search for “best Vim plugins”.

## 3.9 Vim-mode in other programs

Many tools support Vim emulation. The quality varies from good to great; depending on the tool, it may not support the fancier Vim features, but most cover the basics pretty well.

### Shell

If you’re a Bash user, use `set -o vi`. If you use Zsh, `bindkey -v`. For Fish, `fish_vi_key_bindings`. Additionally, no matter what shell you use, you can `export EDITOR=vim`. This is the environment variable used to decide which editor is launched when a program wants to start an editor. For example, `git` will use this editor for commit messages.

### Readline

Many programs use the [GNU Readline](https://tiswww.case.edu/php/chet/readline/rltop.html) library for their command-line interface. Readline supports (basic) Vim emulation too, which can be enabled by adding the following line to the `~/.inputrc` file:

```
set editing-mode vi
```

With this setting, for example, the Python REPL will support Vim bindings.

### Others

There are even vim keybinding extensions for web [browsers](http://vim.wikia.com/wiki/Vim_key_bindings_for_web_browsers) - some popular ones are [Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?hl=en) for Google Chrome and [Tridactyl](https://github.com/tridactyl/tridactyl) for Firefox. You can even get Vim bindings in [Jupyter notebooks](https://github.com/lambdalisue/jupyter-vim-binding). Here is a [long list](https://reversed.top/2016-08-13/big-list-of-vim-like-software) of software with vim-like keybindings.

## 3.10 Advanced Vim

Here are a few examples to show you the power of the editor. We can’t teach you all of these kinds of things, but you’ll learn them as you go. A good heuristic: whenever you’re using your editor and you think “there must be a better way of doing this”, there probably is: look it up online.

### Search and replace

`:s` (substitute) command ([documentation](http://vim.wikia.com/wiki/Search_and_replace)).

-   `%s/foo/bar/g`
    -   replace foo with bar globally in file
-   `%s/\[.*\](\(.*\))/\1/g`
    -   replace named Markdown links with plain URLs

### Multiple windows

-   `:sp` / `:vsp` to split windows
-   Can have multiple views of the same buffer.

### Macros

-   `q{character}` to start recording a macro in register `{character}`
-   `q` to stop recording
-   `@{character}` replays the macro
-   Macro execution stops on error
-   `{number}@{character}` executes a macro {number} times
-   Macros can be recursive
    -   first clear the macro with `q{character}q`
    -   record the macro, with `@{character}` to invoke the macro recursively (will be a no-op until recording is complete)
-   Example: convert xml to json ([file](https://missing.csail.mit.edu/2020/files/example-data.xml))
    -   Array of objects with keys “name” / “email”
    -   Use a Python program?
    -   Use sed / regexes
        -   `g/people/d`
        -   `%s/<person>/{/g`
        -   `%s/<name>\(.*\)<\/name>/"name": "\1",/g`
        -   …
    -   Vim commands / macros
        -   `Gdd`, `ggdd` delete first and last lines
        -   Macro to format a single element (register `e`)
            -   Go to line with `<name>`
            -   `qe^r"f>s": "<ESC>f<C"<ESC>q`
        -   Macro to format a person
            -   Go to line with `<person>`
            -   `qpS{<ESC>j@eA,<ESC>j@ejS},<ESC>q`
        -   Macro to format a person and go to the next person
            -   Go to line with `<person>`
            -   `qq@pjq`
        -   Execute macro until end of file
            -   `999@q`
        -   Manually remove last `,` and add `[` and `]` delimiters

## 3.11 Resources

-   `vimtutor` is a tutorial that comes installed with Vim - if Vim is installed, you should be able to run `vimtutor` from your shell
-   [Vim Adventures](https://vim-adventures.com/) is a game to learn Vim
-   [Vim Tips Wiki](http://vim.wikia.com/wiki/Vim_Tips_Wiki)
-   [Vim Advent Calendar](https://vimways.org/2019/) has various Vim tips
-   [Vim Golf](http://www.vimgolf.com/) is [code golf](https://en.wikipedia.org/wiki/Code_golf), but where the programming language is Vim’s UI
-   [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)
-   [Vim Screencasts](http://vimcasts.org/)
-   [Practical Vim](https://pragprog.com/titles/dnvim2/) (book)

## 3.12 Exercises

1.  Complete `vimtutor`. Note: it looks best in a [80x24](https://en.wikipedia.org/wiki/VT100) (80 columns by 24 lines) terminal window.
2.  Download our [basic vimrc](https://missing.csail.mit.edu/2020/files/vimrc) and save it to `~/.vimrc`. Read through the well-commented file (using Vim!), and observe how Vim looks and behaves slightly differently with the new config.
3.  Install and configure a plugin: [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim).
    1.  Create the plugins directory with `mkdir -p ~/.vim/pack/vendor/start`
    2.  Download the plugin: `cd ~/.vim/pack/vendor/start; git clone https://github.com/ctrlpvim/ctrlp.vim`
    3.  Read the [documentation](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md) for the plugin. Try using CtrlP to locate a file by navigating to a project directory, opening Vim, and using the Vim command-line to start `:CtrlP`.
    4.  Customize CtrlP by adding [configuration](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md#basic-options) to your `~/.vimrc` to open CtrlP by pressing Ctrl-P.
4.  To practice using Vim, re-do the [Demo](https://missing.csail.mit.edu/2020/editors/#demo) from lecture on your own machine.
5.  Use Vim for _all_ your text editing for the next month. Whenever something seems inefficient, or when you think “there must be a better way”, try Googling it, there probably is. If you get stuck, come to office hours or send us an email.
6.  Configure your other tools to use Vim bindings (see instructions above).
7.  Further customize your `~/.vimrc` and install more plugins.
8.  (Advanced) Convert XML to JSON ([example file](https://missing.csail.mit.edu/2020/files/example-data.xml)) using Vim macros. Try to do this on your own, but you can look at the [macros](https://missing.csail.mit.edu/2020/editors/#macros) section above if you get stuck.

# 4. Data Wrangling

本节主要讲的是正则表达式和sed程序的使用，放官方笔记：

Have you ever wanted to **take data in one format and turn it into a different format**? Of course you have! That, in very general terms, is what this lecture is all about. Specifically, massaging data, whether in text or binary format, until you end up with exactly what you wanted.

We’ve already seen some basic data wrangling in past lectures. Pretty much any time you use the `|` operator, you are performing some kind of data wrangling. Consider a command like `journalctl | grep -i intel`. It finds all system log entries that mention Intel (case insensitive). You may not think of it as wrangling data, but it is going from one format (your entire system log) to a format that is more useful to you (just the intel log entries). Most data wrangling is about knowing what tools you have at your disposal, and how to combine them.

Let’s start from the beginning. To wrangle data, we need two things: data to wrangle, and something to do with it. Logs often make for a good use-case, because you often want to investigate things about them, and reading the whole thing isn’t feasible. Let’s figure out who’s trying to log into my server by looking at my server’s log:

```shell
ssh myserver journalctl
```

That’s far too much stuff. Let’s limit it to ssh stuff:

```shell
ssh myserver journalctl | grep sshd
```

Notice that we’re using a pipe to stream a _remote_ file through `grep` on our local computer! `ssh` is magical, and we will talk more about it in the next lecture on the command-line environment. This is still way more stuff than we wanted though. And pretty hard to read. Let’s do better:

```shell
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' | less
```

Why the additional quoting? Well, our logs may be quite large, and it’s wasteful to stream it all to our computer and then do the filtering. Instead, we can do the filtering on the remote server, and then massage the data locally. `less` gives us a “pager” that allows us to scroll up and down through the long output. To save some additional traffic while we debug our command-line, we can even stick the current filtered logs into a file so that we don’t have to access the network while developing:

```shell
$ ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
$ less ssh.log
```

There’s still a lot of noise here. There are _a lot_ of ways to get rid of that, but let’s look at one of the most powerful tools in your toolkit: `sed`.

`sed` is a “stream editor” that builds on top of the old `ed` editor. In it, you basically give short commands for how to modify the file, rather than manipulate its contents directly (although you can do that too). There are tons of commands, but one of the most common ones is `s`: substitution. For example, we can write:

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

What we just wrote was a simple _regular expression_; a powerful construct that lets you match text against patterns. The `s` command is written on the form: `s/REGEX/SUBSTITUTION/`, where `REGEX` is the regular expression you want to search for, and `SUBSTITUTION` is the text you want to substitute matching text with.

(You may recognize this syntax from the “Search and replace” section of our Vim [lecture notes](https://missing.csail.mit.edu/2020/editors/#advanced-vim)! Indeed, Vim uses a syntax for searching and replacing that is similar to `sed`’s substitution command. Learning one tool often helps you become more proficient with others.)

## 4.1 Regular expressions

Regular expressions are common and useful enough that it’s worthwhile to take some time to understand how they work. Let’s start by looking at the one we used above: `/.*Disconnected from /`. Regular expressions are usually (though not always) surrounded by `/`. Most ASCII characters just carry their normal meaning, but some characters have “special” matching behavior. Exactly which characters do what vary somewhat between different implementations of regular expressions, which is a source of great frustration. Very common patterns are:

-   `.` means “any single character” except newline
-   `*` zero or more of the preceding match
-   `+` one or more of the preceding match
-   `[abc]` any one character of `a`, `b`, and `c`
-   `(RX1|RX2)` either something that matches `RX1` or `RX2`
-   `^` the start of the line
-   `$` the end of the line

`sed`’s regular expressions are somewhat weird, and will require you to put a `\` before most of these to give them their special meaning. Or you can pass `-E`.

So, looking back at `/.*Disconnected from /`, we see that it matches any text that starts with any number of characters, followed by the literal string “Disconnected from ”. Which is what we wanted. But

beware, regular expressions are trixy. What if someone tried to log in with the username “Disconnected from”? We’d have:

```shell
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
```

What would we end up with? Well, `*` and `+` are, by default, “greedy”. They will match as much text as they can. So, in the above, we’d end up with just

```shell
46.97.239.16 port 55920 [preauth]
```

Which may not be what we wanted. In some regular expression implementations, you can just suffix `*` or `+` with a `?` to make them non-greedy, but sadly `sed` doesn’t support that. We _could_ switch to perl’s command-line mode though, which _does_ support that construct:

```
perl -pe 's/.*?Disconnected from //'
```

We’ll stick to `sed` for the rest of this, because it’s by far the more common tool for these kinds of jobs. `sed` can also do other handy things like print lines following a given match, do multiple substitutions per invocation, search for things, etc. But we won’t cover that too much here. `sed` is basically an entire topic in and of itself, but there are often better tools.

Okay, so we also have a suffix we’d like to get rid of. How might we do that? It’s a little tricky to match just the text that follows the username, especially if the username can have spaces and such! What we need to do is match the _whole_ line:

```shell
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//'
```

Let’s look at what’s going on with a [regex debugger](https://regex101.com/r/qqbZqh/2). Okay, so the start is still as before. Then, we’re matching any of the “user” variants (there are two prefixes in the logs). Then we’re matching on any string of characters where the username is. Then we’re matching on any single word (`[^ ]+`; any non-empty sequence of non-space characters). Then the word “port” followed by a sequence of digits. Then possibly the suffix `[preauth]`, and then the end of the line.

Notice that with this technique, as username of “Disconnected from” won’t confuse us any more. Can you see why?

There is one problem with this though, and that is that the entire log becomes empty. We want to _keep_ the username after all. For this, we can use “capture groups”. Any text matched by a regex surrounded by parentheses is stored in a numbered capture group. These are available in the substitution (and in some engines, even in the pattern itself!) as `\1`, `\2`, `\3`, etc. So:

```shell
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

As you can probably imagine, you can come up with _really_ complicated regular expressions. For example, here’s an article on how you might match an [e-mail address](https://www.regular-expressions.info/email.html). It’s [not easy](https://emailregex.com/). And there’s [lots of discussion](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982). And people have [written tests](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php). And [test matrices](https://mathiasbynens.be/demo/url-regex). You can even write a regex for determining if a given number [is a prime number](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/).

Regular expressions are notoriously hard to get right, but they are also very handy to have in your toolbox!

## 4.2 Back to data wrangling

Okay, so we now have

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

`sed` can do all sorts of other interesting things, like injecting text (with the `i` command), explicitly printing lines (with the `p` command), selecting lines by index, and lots of other things. Check `man sed`!

Anyway. What we have now gives us a list of all the usernames that have attempted to log in. But this is pretty unhelpful. Let’s look for common ones:

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
```

`sort` will, well, sort its input. `uniq -c` will collapse consecutive lines that are the same into a single line, prefixed with a count of the number of occurrences. We probably want to sort that too and only keep the most common usernames:

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
```

`sort -n` will sort in numeric (instead of lexicographic) order. `-k1,1` means “sort by only the first whitespace-separated column”. The `,n` part says “sort until the `n`th field, where the default is the end of the line. In this _particular_ example, sorting by the whole line wouldn’t matter, but we’re here to learn!

If we wanted the _least_ common ones, we could use `head` instead of `tail`. There’s also `sort -r`, which sorts in reverse order.

Okay, so that’s pretty cool, but what if we’d like these extract only the usernames as a comma-separated list instead of one per line, perhaps for a config file?

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd,
```

If you’re using macOS: note that the command as shown won’t work with the BSD `paste` shipped with macOS. See [exercise 4 from the shell tools lecture](https://missing.csail.mit.edu/2020/shell-tools/#exercises) for more on the difference between BSD and GNU coreutils and instructions for how to install GNU coreutils on macOS.

Let’s start with `paste`: it lets you combine lines (`-s`) by a given single-character delimiter (`-d`; `,` in this case). But what’s this `awk` business?

## 4.3 awk – another editor

`awk` is a programming language that just happens to be really good at processing text streams. There is _a lot_ to say about `awk` if you were to learn it properly, but as with many other things here, we’ll just go through the basics.

First, what does `{print $2}` do? Well, `awk` programs take the form of an optional pattern plus a block saying what to do if the pattern matches a given line. The default pattern (which we used above) matches all lines. Inside the block, `$0` is set to the entire line’s contents, and `$1` through `$n` are set to the `n`th _field_ of that line, when separated by the `awk` field separator (whitespace by default, change with `-F`). In this case, we’re saying that, for every line, print the contents of the second field, which happens to be the username!

Let’s see if we can do something fancier. Let’s compute the number of single-use usernames that start with `c` and end with `e`:

```shell
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l
```

There’s a lot to unpack here. First, notice that we now have a pattern (the stuff that goes before `{...}`). The pattern says that the first field of the line should be equal to 1 (that’s the count from `uniq -c`), and that the second field should match the given regular expression. And the block just says to print the username. We then count the number of lines in the output with `wc -l`.

However, `awk` is a programming language, remember?

```shell
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

`BEGIN` is a pattern that matches the start of the input (and `END` matches the end). Now, the per-line block just adds the count from the first field (although it’ll always be 1 in this case), and then we print it out at the end. In fact, we _could_ get rid of `grep` and `sed` entirely, because `awk` [can do it all](https://backreference.org/2010/02/10/idiomatic-awk/), but we’ll leave that as an exercise to the reader.

## 4.4 Analyzing data

You can do math directly in your shell using `bc`, a calculator that can read from STDIN! For example, add the numbers on each line together by concatenating them together, delimited by `+`:

```shell
 | paste -sd+ | bc -l
```

Or produce more elaborate expressions:

```shell
echo "2*($(data | paste -sd+))" | bc -l
```

You can get stats in a variety of ways. [`st`](https://github.com/nferraz/st) is pretty neat, but if you already have [R](https://www.r-project.org/):

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --no-echo -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
```

R is another (weird) programming language that’s great at data analysis and [plotting](https://ggplot2.tidyverse.org/). We won’t go into too much detail, but suffice to say that `summary` prints summary statistics for a vector, and we created a vector containing the input stream of numbers, so R gives us the statistics we wanted!

If you just want some simple plotting, `gnuplot` is your friend:

```shell
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | gnuplot -p -e 'set boxwidth 0.5; plot "-" using 1:xtic(2) with boxes'
```

## 4.5 Data wrangling to make arguments

Sometimes you want to do data wrangling to find things to install or remove based on some longer list. The data wrangling we’ve talked about so far + `xargs` can be a powerful combo.

For example, as seen in lecture, I can use the following command to uninstall old nightly builds of Rust from my system by extracting the old build names using data wrangling tools and then passing them via `xargs` to the uninstaller:

```shell
rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

## 4.6 Wrangling binary data

So far, we have mostly talked about wrangling textual data, but pipes are just as useful for binary data. For example, we can use ffmpeg to capture an image from our camera, convert it to grayscale, compress it, send it to a remote machine over SSH, decompress it there, make a copy, and then display it.

```shell
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.jpg | env DISPLAY=:0 feh -'
```

## 4.7 Exercises

1.  Take this [short interactive regex tutorial](https://regexone.com/).
2.  Find the number of words (in `/usr/share/dict/words`) that contain at least three `a`s and don’t have a `'s` ending. What are the three most common last two letters of those words? `sed`’s `y` command, or the `tr` program, may help you with case insensitivity. How many of those two-letter combinations are there? And for a challenge: which combinations do not occur?
3.  To do in-place substitution it is quite tempting to do something like `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt`. However this is a bad idea, why? Is this particular to `sed`? Use `man sed` to find out how to accomplish this.
4.  Find your average, median, and max system boot time over the last ten boots. Use `journalctl` on Linux and `log show` on macOS, and look for log timestamps near the beginning and end of each boot. On Linux, they may look something like:

    ```
    Logs begin at ...
    ```   
    
    and
    
    ```
    systemd[577]: Startup finished in ...
    ```
    
    On macOS, [look for](https://eclecticlight.co/2018/03/21/macos-unified-log-3-finding-your-way/):
    
    ```
    === system boot:
    ```
    
    and
    
    ```
    Previous shutdown cause: 5
    ```
    
5.  Look for boot messages that are _not_ shared between your past three reboots (see `journalctl`’s `-b` flag). Break this task down into multiple steps. First, find a way to get just the logs from the past three boots. There may be an applicable flag on the tool you use to extract the boot logs, or you can use `sed '0,/STRING/d'` to remove all lines previous to one that matches `STRING`. Next, remove any parts of the line that _always_ varies (like the timestamp). Then, de-duplicate the input lines and keep a count of each one (`uniq` is your friend). And finally, eliminate any line whose count is 3 (since it _was_ shared among all the boots).
6.  Find an online data set like [this one](https://stats.wikimedia.org/EN/TablesWikipediaZZ.htm), [this one](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1), or maybe one [from here](https://www.springboard.com/blog/free-public-data-sets-data-science-project/). Fetch it using `curl` and extract out just two columns of numerical data. If you’re fetching HTML data, [`pup`](https://github.com/EricChiang/pup) might be helpful. For JSON data, try [`jq`](https://stedolan.github.io/jq/). Find the min and max of one column in a single command, and the difference of the sum of each column in another.

# 5. Command-line Environment

## 5.1 Job Control

sleep是一个程序，可以直接在终端里敲，就是进行睡眠。 在睡眠的时候，我们按ctrl + C，可以终止这个程序，那么原理是什么？实际上，在按ctrl + c的时候，就会给该进程发一个信号，叫做`SIGINT`，也就是中断(interrupt)的信号。因此，我们完全可以在进程中就捕获这个信号，并去做相应的处理。下面写一个程序，这个程序用ctrl + c也停止不了：

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <signal.h>

void handler(int signum){
    printf("I got %d signum, but I don't deal with it.\n", signum);
}

int main(){
    int i = 0;
    signal(SIGINT, handler);
    while(1){
        sleep(1);
        printf("%d", i++);
        fflush(stdout);
        printf("\r\033[k");
    }
    return 0;
}
```

我们在下面这句话中注册了信号处理：

```c
signal(SIGINT, handler);
```

第一个参数是我们要注册什么信号；第二个参数是注册到信号后干嘛。signal函数会将信号自动传递给函数，因此我们在handler函数中就能接收到SIGINT这个信号。下面是这个程序执行的结果，按了ctrl + c之后还是停止不了：

![[Study Log/resources/Pasted image 20221219213915.png]]

> 这个程序的更多细节可以在[[Article/story_on_program#2022-12-19|我的开发日记]]里看到。

那么问题来了，如何停止呢？答案是按ctrl + \\，这其实是发送了一个SIGQUIT信号，而这个程序不能处理SIGQUIT信号，因此终止了。

---

当我们想停止程序的时候，可以按ctrl + z：

![[Study Log/resources/Pasted image 20221219214448.png]]

这个时候进程其实并没有结束，那么在哪里呢？我们通过`jobs`命令可以看到：

![[Study Log/resources/Pasted image 20221219214540.png]]

可以看到它的状态是stopped，所以并不是真正被杀死了，只是被暂停了。那么如何让它继续呢？在说这个之前，先说一件事，就是有些进程，比如在一个很大的文件夹里找一个文件。这种进程会持续很久，我们如果不想让它干扰我们的操作，就要让它在后台运行。如何做到呢？像这样： ^acf89f

```shell
sleep 1000 &
```

只需要在最后加上一个`&`符号，这个进程就会在后台执行了。

![[Study Log/resources/Pasted image 20221219214848.png]]

然后，如果我要让这个正在后台进程的程序暂停，该如何做到呢？这时就可以用`kill`指令。我们早就知道kill指令可以发送信号给进程，所以只需要：

![[Study Log/resources/Pasted image 20221219215023.png]]

这样我们就看到第二个进程被暂停了。接下来就回到了[[#^acf89f|前面的问题]]：如何让它继续？这里就涉及到了两种启动任务的命令：

* `bg` - background
* `fg` - foreground

![[Study Log/resources/Pasted image 20221219215310.png]]

---

需要强调的是SIGHUP这个信号，这表示hung up，也就是挂断的时候的信号。当这个进程的终端被关掉时，这个进程就会收到这个信号，那么自然这个进程也会结束。我们可以做个实验：给一个进程发送SIGHUP，**之后这个进程在很短的时间内会处于hungup状态，然后马上就死了**：

![[Study Log/resources/Pasted image 20221219215818.png]]

然而有一种命令可以无视这种信号，这样在终端挂掉的时候，这个进程还是会继续进行下去。这种进程是使用nohup启动的：

```shell
nohup sleep 1001 &
```

![[Study Log/resources/Pasted image 20221219220144.png]]

> 这个时候，向这个进程发送SIGHUP：
>  * 如果这个进程正在运行，它会无视SIGHUP，继续运行；
>  * 如果这个进程已经停止，它不但不会死掉，反而还是会继续运行。

## 5.2 Terminal Multiplexer

### 5.2.1 Session

将终端变成多窗口！超级炫酷！目前最常用的Multiplexer就是tmux。我们安装好之后，只需要输入下面的命令：

```shell
tmux
```

就可以开启一个新的终端(好像是这样)：

![[Study Log/resources/Pasted image 20230111141316.png]]

但是，实际上我们只是新建了一个session。每使用一次`tmux`命令，都会新建一个session。我们可以把它当成一个新的终端来使用：

![[Study Log/resources/Pasted image 20230111141354.png]]

从当前的session**暂时**返回到主终端，我们可以这样做：先按`Ctrl + b`，然后再按一下`d`。我们也能发现，在主终端上出现了下面的字样：

![[Study Log/resources/Pasted image 20230111141421.png]]

`detach`就表示暂时断开连接，attach的反义词。因此我们又会到了主终端。但是，那个session中的程序实际上还是在运行的，我们可以使用如下命令返回到那个session：

```shell
tmux a
```

我们也可以使用下面的命令来列出当前新建了多少个session：

```shell
tmux ls
```

![[Study Log/resources/Pasted image 20230111141445.png]]

还可以给新建的session起一个名字：

```shell
tmux new -t haha
```

这样我们就新建了一个名叫haha的session。和之前默认的一起，我们可以看到它们的情况：

![[Study Log/resources/Pasted image 20230111141522.png]]

有了多个session，还可以通过session的编号和名字去attach它们：

```shell
tmux a -t 0
tmux a -t haha
```

### 5.2.2 Window

说完了session，下面看看每个session里都能干什么。实际上，session中的操作和vim多窗口有点像。我们可以按`Ctrl + b`，然后再按一下`c`来create一个新的window：

在屏幕的最下面能看到当前处于哪一个window。下一个问题显而易见了：怎么在Window之间切换呢？使用`Ctrl + b`和`p`来切换到上一个(previous)window；使用`Ctrl + b`和`n`来切换到下一个(next)window。

session可以改名字，window当然也可以改！在当前的window下，使用`Ctrl + b`和`,`来给当前的window改名字：

![[Study Log/resources/Pasted image 20230111141640.png]]

> 第三个window被改成了spreadzhao。

每个window都有编号，从0开始。我们也可以使用`Ctrl + b`和数字来定位到那个window。

### 5.2.3 Pane

到目前为止，我们也只是实现了类似浏览器标签那种样子，但是真正的分屏多窗口还没有实现。而pane就是用来做这件事的。每一个window都可以新建若干个pane，使用`Ctrl + b`和`"`在当前window下分两屏：

![[Study Log/resources/Pasted image 20230111141824.png]]

可以看到，这样是垂直切割。那么怎么水平切割呢？使用`Ctrl + b`和`%`就可以了：

![[Study Log/resources/Pasted image 20230111141856.png]]

和window一样，如何在pane之前来回切换呢？使用`Ctrl + b`和`方向键`就可以了。如果我觉得这种排版不合适，可以使用`Ctrl + b`和`space`。多用几次，它会给你不同的排版：

![[Study Log/resources/Pasted image 20230111141917.png]]

比如我在某一个pane里打开vim，发现窗口太小了，怎么放大呢？使用`Ctrl + b`和z来放大(zoom)当前窗口。如果我用完了，再来一遍就可以缩回去。

![[Study Log/resources/Pasted image 20230111142035.png]]

![[Study Log/resources/Pasted image 20230111142101.png]]

**最后来说一下关闭。不管是pane，window还是session，统一使用`Ctrl + d`来关闭**。

## 5.3 Dotfiles

dotfile实际上就是那些前面带点的文件：`.bashrc`，`.vimrc`，`.gitignore`等等。这些都是配置文件，而通常它们都在`~`目录下。这些东西就是每次打开一个终端的时候都会加载好的，因此这些配置虽然不写到环境变量里，但是却随时都能生效。

我们有时候可能会有这样的需求：某一个配置文件在一个仓库里用，但是这个仓库的配置和我本机的配置不一样。而且这个配置文件还总会变。那我该咋办？如果每写一个项目就要改一下配置文件那也太麻烦了！所以才有了dotfile repository的概念。我们将用户目录改成下面的结构：

![[Study Log/resources/Pasted image 20230111133634.png]]

这两个最基础的`.bashrc`和`.vimrc`里并没有内容，只是有链接。它们指向不同的配置文件。使用这种方式，当我们想要切换文件的时候，只需要把这个指向的链接修改了就可以了。

#idea 其实这种思想和项目里经常看到的多重include很像。我经常在项目里看到，某一个.h文件打开之后就一句话，就是include了许多.h。这种做法其实也是为了便于切换引用的库。

## 5.4 Lecture Notes

In this lecture we will go through several ways in which you can improve your workflow when using the shell. We have been working with the shell for a while now, but we have mainly focused on executing different commands. We will now see how to run several processes at the same time while keeping track of them, how to stop or pause a specific process and how to make a process run in the background.

We will also learn about different ways to improve your shell and other tools, by defining aliases and configuring them using dotfiles. Both of these can help you save time, e.g. by using the same configurations in all your machines without having to type long commands. We will look at how to work with remote machines using SSH.

### 5.4.1 Job Control

In some cases you will need to interrupt a job while it is executing, for instance if a command is taking too long to complete (such as a `find` with a very large directory structure to search through). Most of the time, you can do `Ctrl-C` and the command will stop. But how does this actually work and why does it sometimes fail to stop the process?

#### 5.4.1.1Killing a process

Your shell is using a UNIX communication mechanism called a _signal_ to communicate information to the process. When a process receives a signal it stops its execution, deals with the signal and potentially changes the flow of execution based on the information that the signal delivered. For this reason, signals are _software interrupts_.

In our case, when typing `Ctrl-C` this prompts the shell to deliver a `SIGINT` signal to the process.

Here’s a minimal example of a Python program that captures `SIGINT` and ignores it, no longer stopping. To kill this program we can now use the `SIGQUIT` signal instead, by typing `Ctrl-\`.

```shell
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

Here’s what happens if we send `SIGINT` twice to this program, followed by `SIGQUIT`. Note that `^` is how `Ctrl` is displayed when typed in the terminal.

```shell
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

While `SIGINT` and `SIGQUIT` are both usually associated with terminal related requests, a more generic signal for asking a process to exit gracefully is the `SIGTERM` signal. To send this signal we can use the [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html) command, with the syntax `kill -TERM <PID>`.

#### 5.4.1.2 Pausing and backgrounding processes

Signals can do other things beyond killing a process. For instance, `SIGSTOP` pauses a process. In the terminal, typing `Ctrl-Z` will prompt the shell to send a `SIGTSTP` signal, short for Terminal Stop (i.e. the terminal’s version of `SIGSTOP`).

We can then continue the paused job in the foreground or in the background using [`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) or [`bg`](http://man7.org/linux/man-pages/man1/bg.1p.html), respectively.

The [`jobs`](https://www.man7.org/linux/man-pages/man1/jobs.1p.html) command lists the unfinished jobs associated with the current terminal session. You can refer to those jobs using their pid (you can use [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) to find that out). More intuitively, you can also refer to a process using the percent symbol followed by its job number (displayed by `jobs`). To refer to the last backgrounded job you can use the `$!` special parameter.

One more thing to know is that the `&` suffix in a command will run the command in the background, giving you the prompt back, although it will still use the shell’s STDOUT which can be annoying (use shell redirections in that case).

To background an already running program you can do `Ctrl-Z` followed by `bg`. Note that backgrounded processes are still children processes of your terminal and will die if you close the terminal (this will send yet another signal, `SIGHUP`). To prevent that from happening you can run the program with [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html) (a wrapper to ignore `SIGHUP`), or use `disown` if the process has already been started. Alternatively, you can use a terminal multiplexer as we will see in the next section.

Below is a sample session to showcase some of these concepts.

```shell
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ bg %1
[1]  - 18653 continued  sleep 1000

$ jobs
[1]  - running    sleep 1000
[2]  + running    nohup sleep 2000

$ kill -STOP %1
[1]  + 18653 suspended (signal)  sleep 1000

$ jobs
[1]  + suspended (signal)  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ jobs
[2]  + running    nohup sleep 2000

$ kill -SIGHUP %2

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000

$ jobs

```

A special signal is `SIGKILL` since it cannot be captured by the process and it will always terminate it immediately. However, it can have bad side effects such as leaving orphaned children processes.

You can learn more about these and other signals [here](https://en.wikipedia.org/wiki/Signal_(IPC)) or typing [`man signal`](https://www.man7.org/linux/man-pages/man7/signal.7.html) or `kill -l`.

### 5.4.2 Terminal Multiplexers

When using the command line interface you will often want to run more than one thing at once. For instance, you might want to run your editor and your program side by side. Although this can be achieved by opening new terminal windows, using a terminal multiplexer is a more versatile solution.

Terminal multiplexers like [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) allow you to multiplex terminal windows using panes and tabs so you can interact with multiple shell sessions. Moreover, terminal multiplexers let you detach a current terminal session and reattach at some point later in time. This can make your workflow much better when working with remote machines since it avoids the need to use `nohup` and similar tricks.

The most popular terminal multiplexer these days is [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html). `tmux` is highly configurable and by using the associated keybindings you can create multiple tabs and panes and quickly navigate through them.

`tmux` expects you to know its keybindings, and they all have the form `<C-b> x` where that means (1) press `Ctrl+b`, (2) release `Ctrl+b`, and then (3) press `x`. `tmux` has the following hierarchy of objects:

-   **Sessions** - a session is an independent workspace with one or more windows
    -   `tmux` starts a new session.
    -   `tmux new -s NAME` starts it with that name.
    -   `tmux ls` lists the current sessions
    -   Within `tmux` typing `<C-b> d` detaches the current session
    -   `tmux a` attaches the last session. You can use `-t` flag to specify which
-   **Windows** - Equivalent to tabs in editors or browsers, they are visually separate parts of the same session
    -   `<C-b> c` Creates a new window. To close it you can just terminate the shells doing `<C-d>`
    -   `<C-b> N` Go to the _N_ th window. Note they are numbered
    -   `<C-b> p` Goes to the previous window
    -   `<C-b> n` Goes to the next window
    -   `<C-b> ,` Rename the current window
    -   `<C-b> w` List current windows
-   **Panes** - Like vim splits, panes let you have multiple shells in the same visual display.
    -   `<C-b> "` Split the current pane horizontally
    -   `<C-b> %` Split the current pane vertically
    -   `<C-b> <direction>` Move to the pane in the specified _direction_. Direction here means arrow keys.
    -   `<C-b> z` Toggle zoom for the current pane
    -   `<C-b> [` Start scrollback. You can then press `<space>` to start a selection and `<enter>` to copy that selection.
    -   `<C-b> <space>` Cycle through pane arrangements.

For further reading, [here](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) is a quick tutorial on `tmux` and [this](http://linuxcommand.org/lc3_adv_termmux.php) has a more detailed explanation that covers the original `screen` command. You might also want to familiarize yourself with [`screen`](https://www.man7.org/linux/man-pages/man1/screen.1.html), since it comes installed in most UNIX systems.

### 5.4.3 Aliases

It can become tiresome typing long commands that involve many flags or verbose options. For this reason, most shells support _aliasing_. A shell alias is a short form for another command that your shell will replace automatically for you. For instance, an alias in bash has the following structure:

```shell
alias alias_name="command_to_alias arg1 arg2"
```

Note that there is no space around the equal sign `=`, because [`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html) is a shell command that takes a single argument.

Aliases have many convenient features:

```shell
# Make shorthands for common flags
alias ll="ls -lh"

# Save a lot of typing for common commands
alias gs="git status"
alias gc="git commit"
alias v="vim"

# Save you from mistyping
alias sl=ls

# Overwrite existing commands for better defaults
alias mv="mv -i"           # -i prompts before overwrite
alias mkdir="mkdir -p"     # -p make parent dirs as needed
alias df="df -h"           # -h prints human readable format

# Alias can be composed
alias la="ls -A"
alias lla="la -l"

# To ignore an alias run it prepended with \
\ls
# Or disable an alias altogether with unalias
unalias la

# To get an alias definition just call it with alias
alias ll
# Will print ll='ls -lh'
```

Note that aliases do not persist shell sessions by default. To make an alias persistent you need to include it in shell startup files, like `.bashrc` or `.zshrc`, which we are going to introduce in the next section.

### 5.4.4 Dotfiles

Many programs are configured using plain-text files known as _dotfiles_ (because the file names begin with a `.`, e.g. `~/.vimrc`, so that they are hidden in the directory listing `ls` by default).

Shells are one example of programs configured with such files. On startup, your shell will read many files to load its configuration. Depending on the shell, whether you are starting a login and/or interactive the entire process can be quite complex. [Here](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html) is an excellent resource on the topic.

For `bash`, editing your `.bashrc` or `.bash_profile` will work in most systems. Here you can include commands that you want to run on startup, like the alias we just described or modifications to your `PATH` environment variable. In fact, many programs will ask you to include a line like `export PATH="$PATH:/path/to/program/bin"` in your shell configuration file so their binaries can be found.

Some other examples of tools that can be configured through dotfiles are:

-   `bash` - `~/.bashrc`, `~/.bash_profile`
-   `git` - `~/.gitconfig`
-   `vim` - `~/.vimrc` and the `~/.vim` folder
-   `ssh` - `~/.ssh/config`
-   `tmux` - `~/.tmux.conf`

How should you organize your dotfiles? They should be in their own folder, under version control, and **symlinked** into place using a script. This has the benefits of:

-   **Easy installation**: if you log in to a new machine, applying your customizations will only take a minute.
-   **Portability**: your tools will work the same way everywhere.
-   **Synchronization**: you can update your dotfiles anywhere and keep them all in sync.
-   **Change tracking**: you’re probably going to be maintaining your dotfiles for your entire programming career, and version history is nice to have for long-lived projects.

What should you put in your dotfiles? You can learn about your tool’s settings by reading online documentation or [man pages](https://en.wikipedia.org/wiki/Man_page). Another great way is to search the internet for blog posts about specific programs, where authors will tell you about their preferred customizations. Yet another way to learn about customizations is to look through other people’s dotfiles: you can find tons of [dotfiles repositories](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories) on Github — see the most popular one [here](https://github.com/mathiasbynens/dotfiles) (we advise you not to blindly copy configurations though). [Here](https://dotfiles.github.io/) is another good resource on the topic.

All of the class instructors have their dotfiles publicly accessible on GitHub: [Anish](https://github.com/anishathalye/dotfiles), [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/jjgo/dotfiles).

#### 5.4.4.1 Portability

A common pain with dotfiles is that the configurations might not work when working with several machines, e.g. if they have different operating systems or shells. Sometimes you also want some configuration to be applied only in a given machine.

There are some tricks for making this easier. If the configuration file supports it, use the equivalent of if-statements to apply machine specific customizations. For example, your shell could have something like:

```shell
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# Check before using shell-specific features
if [[ "$SHELL" == "zsh" ]]; then {do_something}; fi

# You can also make it machine-specific
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi
```

If the configuration file supports it, make use of includes. For example, a `~/.gitconfig` can have a setting:

```shell
[include]
    path = ~/.gitconfig_local
```

And then on each machine, `~/.gitconfig_local` can contain machine-specific settings. You could even track these in a separate repository for machine-specific settings.

This idea is also useful if you want different programs to share some configurations. For instance, if you want both `bash` and `zsh` to share the same set of aliases you can write them under `.aliases` and have the following block in both:

```shell
# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

### 5.4.5 Remote Machines

It has become more and more common for programmers to use remote servers in their everyday work. If you need to use remote servers in order to deploy backend software or you need a server with higher computational capabilities, you will end up using a Secure Shell (SSH). As with most tools covered, SSH is highly configurable so it is worth learning about it.

To `ssh` into a server you execute a command as follows

```shell
ssh foo@bar.mit.edu
```

Here we are trying to ssh as user `foo` in server `bar.mit.edu`. The server can be specified with a URL (like `bar.mit.edu`) or an IP (something like `foobar@192.168.1.42`). Later we will see that if we modify ssh config file you can access just using something like `ssh bar`.

#### 5.4.5.1 Executing commands

An often overlooked feature of `ssh` is the ability to run commands directly. `ssh foobar@server ls` will execute `ls` in the home folder of foobar. It works with pipes, so `ssh foobar@server ls | grep PATTERN` will grep locally the remote output of `ls` and `ls | ssh foobar@server grep PATTERN` will grep remotely the local output of `ls`.

#### 5.4.5.2 SSH Keys

Key-based authentication exploits public-key cryptography to prove to the server that the client owns the secret private key without revealing the key. This way you do not need to reenter your password every time. Nevertheless, the private key (often `~/.ssh/id_rsa` and more recently `~/.ssh/id_ed25519`) is effectively your password, so treat it like so.

##### 5.4.5.2.1 Key generation

To generate a pair you can run [`ssh-keygen`](https://www.man7.org/linux/man-pages/man1/ssh-keygen.1.html).

```shell
ssh-keygen -o -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

You should choose a passphrase, to avoid someone who gets hold of your private key to access authorized servers. Use [`ssh-agent`](https://www.man7.org/linux/man-pages/man1/ssh-agent.1.html) or [`gpg-agent`](https://linux.die.net/man/1/gpg-agent) so you do not have to type your passphrase every time.

If you have ever configured pushing to GitHub using SSH keys, then you have probably done the steps outlined [here](https://help.github.com/articles/connecting-to-github-with-ssh/) and have a valid key pair already. To check if you have a passphrase and validate it you can run `ssh-keygen -y -f /path/to/key`.

##### 5.4.5.2.2 Key based authentication

`ssh` will look into `.ssh/authorized_keys` to determine which clients it should let in. To copy a public key over you can use:

```shell
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'
```

A simpler solution can be achieved with `ssh-copy-id` where available:

```shell
ssh-copy-id -i .ssh/id_ed25519 foobar@remote
```

#### 5.4.5.2 Copying files over SSH

There are many ways to copy files over ssh:

-   `ssh+tee`, the simplest is to use `ssh` command execution and STDIN input by doing `cat localfile | ssh remote_server tee serverfile`. Recall that [`tee`](https://www.man7.org/linux/man-pages/man1/tee.1.html) writes the output from STDIN into a file.
-   [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) when copying large amounts of files/directories, the secure copy `scp` command is more convenient since it can easily recurse over paths. The syntax is `scp path/to/local_file remote_host:path/to/remote_file`
-   [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) improves upon `scp` by detecting identical files in local and remote, and preventing copying them again. It also provides more fine grained control over symlinks, permissions and has extra features like the `--partial` flag that can resume from a previously interrupted copy. `rsync` has a similar syntax to `scp`.

#### 5.4.5.3 Port Forwarding

In many scenarios you will run into software that listens to specific ports in the machine. When this happens in your local machine you can type `localhost:PORT` or `127.0.0.1:PORT`, but what do you do with a remote server that does not have its ports directly available through the network/internet?.

This is called _port forwarding_ and it comes in two flavors: Local Port Forwarding and Remote Port Forwarding (see the pictures for more details, credit of the pictures from [this StackOverflow post](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot)).

**Local Port Forwarding**

![[Study Log/resources/Pasted image 20230111143410.png]]

**Remote Port Forwarding**

![[Study Log/resources/Pasted image 20230111143424.png]]

The most common scenario is local port forwarding, where a service in the remote machine listens in a port and you want to link a port in your local machine to forward to the remote port. For example, if we execute `jupyter notebook` in the remote server that listens to the port `8888`. Thus, to forward that to the local port `9999`, we would do `ssh -L 9999:localhost:8888 foobar@remote_server` and then navigate to `locahost:9999` in our local machine.

#### 5.4.5.4 SSH Configuration

We have covered many many arguments that we can pass. A tempting alternative is to create shell aliases that look like

```shell
alias my_server="ssh -i ~/.id_ed25519 --port 2222 -L 9999:localhost:8888 foobar@remote_server
```

However, there is a better alternative using `~/.ssh/config`.

```shell
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# Configs can also take wildcards
Host *.mit.edu
    User foobaz
```

An additional advantage of using the `~/.ssh/config` file over aliases is that other programs like `scp`, `rsync`, `mosh`, &c are able to read it as well and convert the settings into the corresponding flags.

Note that the `~/.ssh/config` file can be considered a dotfile, and in general it is fine for it to be included with the rest of your dotfiles. However, if you make it public, think about the information that you are potentially providing strangers on the internet: addresses of your servers, users, open ports, &c. This may facilitate some types of attacks so be thoughtful about sharing your SSH configuration.

Server side configuration is usually specified in `/etc/ssh/sshd_config`. Here you can make changes like disabling password authentication, changing ssh ports, enabling X11 forwarding, &c. You can specify config settings on a per user basis.

#### 5.4.5.5 Miscellaneous

A common pain when connecting to a remote server are disconnections due to your computer shutting down, going to sleep, or changing networks. Moreover if one has a connection with significant lag using ssh can become quite frustrating. [Mosh](https://mosh.org/), the mobile shell, improves upon ssh, allowing roaming connections, intermittent connectivity and providing intelligent local echo.

Sometimes it is convenient to mount a remote folder. [sshfs](https://github.com/libfuse/sshfs) can mount a folder on a remote server locally, and then you can use a local editor.

### 5.4.6 Shells & Frameworks

During shell tool and scripting we covered the `bash` shell because it is by far the most ubiquitous shell and most systems have it as the default option. Nevertheless, it is not the only option.

For example, the `zsh` shell is a superset of `bash` and provides many convenient features out of the box such as:

-   Smarter globbing, `**`
-   Inline globbing/wildcard expansion
-   Spelling correction
-   Better tab completion/selection
-   Path expansion (`cd /u/lo/b` will expand as `/usr/local/bin`)

**Frameworks** can improve your shell as well. Some popular general frameworks are [prezto](https://github.com/sorin-ionescu/prezto) or [oh-my-zsh](https://ohmyz.sh/), and smaller ones that focus on specific features such as [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) or [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search). Shells like [fish](https://fishshell.com/) include many of these user-friendly features by default. Some of these features include:

-   Right prompt
-   Command syntax highlighting
-   History substring search
-   manpage based flag completions
-   Smarter autocompletion
-   Prompt themes

One thing to note when using these frameworks is that they may slow down your shell, especially if the code they run is not properly optimized or it is too much code. You can always profile it and disable the features that you do not use often or value over speed.

### 5.4.7 Terminal Emulators

Along with customizing your shell, it is worth spending some time figuring out your choice of **terminal emulator** and its settings. There are many many terminal emulators out there (here is a [comparison](https://anarc.at/blog/2018-04-12-terminal-emulators-1/)).

Since you might be spending hundreds to thousands of hours in your terminal it pays off to look into its settings. Some of the aspects that you may want to modify in your terminal include:

-   Font choice
-   Color Scheme
-   Keyboard shortcuts
-   Tab/Pane support
-   Scrollback configuration
-   Performance (some newer terminals like [Alacritty](https://github.com/jwilm/alacritty) or [kitty](https://sw.kovidgoyal.net/kitty/) offer GPU acceleration).

### 5.4.8 Exercises

#### 5.4.8.1 Job control

1.  From what we have seen, we can use some `ps aux | grep` commands to get our jobs’ pids and then kill them, but there are better ways to do it. Start a `sleep 10000` job in a terminal, background it with `Ctrl-Z` and continue its execution with `bg`. Now use [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) to find its pid and [`pkill`](http://man7.org/linux/man-pages/man1/pgrep.1.html) to kill it without ever typing the pid itself. (Hint: use the `-af` flags).
    
2.  Say you don’t want to start a process until another completes. How would you go about it? In this exercise, our limiting process will always be `sleep 60 &`. One way to achieve this is to use the [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html) command. Try launching the sleep command and having an `ls` wait until the background process finishes.
    
    However, this strategy will fail if we start in a different bash session, since `wait` only works for child processes. One feature we did not discuss in the notes is that the `kill` command’s exit status will be zero on success and nonzero otherwise. `kill -0` does not send a signal but will give a nonzero exit status if the process does not exist. Write a bash function called `pidwait` that takes a pid and waits until the given process completes. You should use `sleep` to avoid wasting CPU unnecessarily.
    

#### 5.4.8.2 Terminal multiplexer

1.  Follow this `tmux` [tutorial](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) and then learn how to do some basic customizations following [these steps](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/).

#### 5.4.8.3 Aliases

1.  Create an alias `dc` that resolves to `cd` for when you type it wrongly.
    
2.  Run `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` to get your top 10 most used commands and consider writing shorter aliases for them. Note: this works for Bash; if you’re using ZSH, use `history 1` instead of just `history`.
    

#### 5.4.8.4 Dotfiles

Let’s get you up to speed with dotfiles.

1.  Create a folder for your dotfiles and set up version control.
2.  Add a configuration for at least one program, e.g. your shell, with some customization (to start off, it can be something as simple as customizing your shell prompt by setting `$PS1`).
3.  Set up a method to install your dotfiles quickly (and without manual effort) on a new machine. This can be as simple as a shell script that calls `ln -s` for each file, or you could use a [specialized utility](https://dotfiles.github.io/utilities/).
4.  Test your installation script on a fresh virtual machine.
5.  Migrate all of your current tool configurations to your dotfiles repository.
6.  Publish your dotfiles on GitHub.

#### 5.4.8.5 Remote Machines

Install a Linux virtual machine (or use an already existing one) for this exercise. If you are not familiar with virtual machines check out [this](https://hibbard.eu/install-ubuntu-virtual-box/) tutorial for installing one.

1.  Go to `~/.ssh/` and check if you have a pair of SSH keys there. If not, generate them with `ssh-keygen -o -a 100 -t ed25519`. It is recommended that you use a password and use `ssh-agent` , more info [here](https://www.ssh.com/ssh/agent).
2.  Edit `.ssh/config` to have an entry as follows
    
    ```shell
     Host vm
         User username_goes_here
         HostName ip_goes_here
         IdentityFile ~/.ssh/id_ed25519
         LocalForward 9999 localhost:8888
    ```
    
3.  Use `ssh-copy-id vm` to copy your ssh key to the server.
4.  Start a webserver in your VM by executing `python -m http.server 8888`. Access the VM webserver by navigating to `http://localhost:9999` in your machine.
5.  Edit your SSH server config by doing `sudo vim /etc/ssh/sshd_config` and disable password authentication by editing the value of `PasswordAuthentication`. Disable root login by editing the value of `PermitRootLogin`. Restart the `ssh` service with `sudo service sshd restart`. Try sshing in again.
6.  (Challenge) Install [`mosh`](https://mosh.org/) in the VM and establish a connection. Then disconnect the network adapter of the server/VM. Can mosh properly recover from it?
7.  (Challenge) Look into what the `-N` and `-f` flags do in `ssh` and figure out a command to achieve background port forwarding.

# 6. Version Control

你是否有过这样的经历：

![[Study Log/resources/Pasted image 20230112134327.png|300]]

在我们初学git的时候，项目出现错误了。那以我当时的水平，能做的就是：删掉现在的项目，clone原来没错的版本。然而经过本章的学习，就不会再做这样的事情了。因为git强大到很轻松就能应付这类问题。

git使用的是Directed Asyclic Graph(有向无环图)来管理版本信息。从简单的角度来讲，比如我整个仓库首次上传的时候，有一个最初的状态：

![[Excalidraw/Drawing 2023-01-12 14.11.16.excalidraw|50]]

那么当我开发出下一个版本时，这个新状态就会指向原来的版本，也就是它的父亲：

![[Excalidraw/Drawing 2023-01-12 14.12.17.excalidraw|200]]

接下来我们遇到了一些困难：当前开发出来的这个新版本有一些bug要修；而于此同时，我还有一些新的特性要加进去。这个时候，git就可以同时产生两个状态，它们都指向这个新版本。

![[Excalidraw/Drawing 2023-01-12 14.14.17.excalidraw]]

之后，我的新特性添加完了，同时我的bug也修完了。那我当然要把它俩合为一体。这个时候又出现一个新状态，它同时指向这两个分支：

![[Excalidraw/Drawing 2023-01-12 14.16.16.excalidraw|500]]



