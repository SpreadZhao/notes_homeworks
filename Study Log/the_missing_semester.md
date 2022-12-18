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