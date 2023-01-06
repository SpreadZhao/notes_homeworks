欢迎来到Spread Zhao的笔记仓库！**这里是我所有知识的积累，并且会持续更新下去**！有关每个文件夹的定义，可以到各个文件夹的`README`文件去查看，这里给出所有的课堂笔记：

```dataview
table description
from "/"
where category="inter_class"
```

> *该查询使用了DataView插件，版本在下面。*

**==对于仓库内部细节的更新，我会以日记的形式更新在这下面==**，每一天的日记都会以横线分割。另外，介绍一下这个仓库中常用的标签，**它们有可能会并列出现**：

#TODO 这是我还没有完成的任务，可以是对仓库的更新任务；也可以是学习过程中立下的目标。这个标签下面紧跟着的就是各种任务的集合，就是`- [x]`这种形式。

#question 这是在任何时刻遇到的各种问题。有时是老师抛出来的，有时是我自己想到的；如果已经有了答案，会在下面直接给出。

#example 这是课上老师给的例题。

#homework 这是嵌在笔记中的作业题。通常是单独的一个标题，只是会在下面加上这样的标签。

#poe Point On Exam。这是考试有可能出现的考点

#idea 这是我突发奇想的点子，或者是一些我认为比较重要的东西。

#keypoint 关键点，任何可能的重点/难点都会使用此标签。

---

# 2022-10-17

本仓库使用obsidian搭建，并且融合进了HappySE的仓库。关于HappySE仓库的作者信息如下：

[[Happy-SE-in-XDU-master/README]]

本仓库使用的obsidian第三方插件如下：

![[Pasted image 20221017125039.png]]

如果以后有更新也会在此文件中说明。由于我将`.obsidian`文件夹放在了`.gitignore`中(这是为了同步的方便)，所以诸如第三方插件这样的配置需要靠自己去搞定\~\~

**我是SpreadZhao，来自西电的一名码农。**

---

# 2022-10-18

新增了插件：

![[Pasted image 20221018121850.png]]

---

# 2022-10-19 

导出pdf的样式丑的要死，怀疑是1.0版本升级之后改成这个样子。解决办法参见[[software_qa#pdf导出|这里]]，但是解决成这样还是不太好看。

---

# 2022-10-21

增加了插件：

![[Pasted image 20221021211952.png]]

另外DataView插件要进行如下设置：

![[Pasted image 20221022185115.png]]

不然会和某些代码段冲突。

---

# 2022-11-10

由于引入了yaml模板，所以在Obsidian中要进行如下设置应用模板：

![[Pasted image 20221110232507.png]]

---

# 2022-11-21

增加了插件，下面给出目前的所有插件：

![[resources/Pasted image 20221121123025.png]]

另外给出文件链接的新设置，今天之后插入的连接也都是这种格式。

#TODO Update Link

- [ ] 将所有的图片都改成这种格式

![[resources/Pasted image 20221121161025.png]]

---

# 2022-11-24

如何使用本仓库？其实非常简单。

首先，去下面的网址下载obsidian：

[Obsidian](https://obsidian.md/)

![[resources/Pasted image 20221124143144.png]]

点击`Get Obsidian fo Windows`即可。下载安装完之后，会打开这样一个窗口：

![[resources/Pasted image 20221124143223.png]]

一个仓库说白了就是一个文件夹。那么我们只需要把我的仓库下下来然后选择打开本地仓库，就欧克了！

首先来到我的仓库地址：

[notes_homeworks: 之前的笔记仓库已经弃用，从Typora转为了Obsidian (gitee.com)](https://gitee.com/spreadzhao/notes_homeworks)

在里面你能够找到这个按钮：

![[resources/Pasted image 20221124143355.png]]

点击这个复制，这就是本项目的仓库地址。我们用这个地址把仓库下载下来。

去这个地址下载git工具：

[Git - Downloads (git-scm.com)](https://git-scm.com/downloads)

![[resources/Pasted image 20221124143455.png]]

点击这个windows，之后点击下面的按钮：

![[resources/Pasted image 20221124143551.png]]

下载完安装即可。安装的过程可能会有一点漫长，不过不要紧，全部默认就ok了。在安装完之后，在任意一个文件夹下右键都能看到这个：

![[resources/Pasted image 20221124143649.png]]

`Git Bash Here`就是我们最常用的功能了。我们在随便一个文件夹里点击它，会出现一个类似cmd的窗口：

![[resources/Pasted image 20221124143751.png]]

我们在这里下载我的仓库。输入如下命令：

```shell
git clone https://gitee.com/spreadzhao/notes_homeworks.git
```

这里的地址就是之前复制的仓库地址。**但是注意粘贴的方式不一样**，在gitbash中粘贴的快捷键是`SHIFT + INSERT` ，而不是`CTRL + V`。如果找不到的话鼠标右键粘贴也行。

等待一段时间，仓库就下载好了，之后我们就要在obsidian中打开这个仓库：

![[resources/Pasted image 20221124144100.png]]

点击打开后，选择刚刚下载好的文件夹，就可以使用了！！！

另外，如果仓库有更新，就来到下载的仓库目录下，右键打开`Git Bash Here`，输入如下命令：

```shell
git pull
```

这样就会自动将仓库更新成最新的。

> 最好自己注册一个gitee的账号，有可能进行git操作的时候会让你登录。

# 2022-12-29

更新了插件：

![[resources/Pasted image 20221229144203.png]]