---
title:
  - "2.6 Scheduling: Proportional Share"
order: "7"
---
[[Study Log/os_study/0_ostep_index|Return to Index]]

## 2.6 Scheduling: Proportional Share

和上一节一样，我们也要讨论一种调度策略。但是和MLFQ的出发点是不一样的。之前我们讨论过两个指标：Turnaround Time和Response Time，分别为了衡量被服务的充分程度和响应的速度。但是本节讨论的调度策略，目的不太一样。和它的名字一样，Proportional (成比例的) Share，有时候也叫Fair Share。目的是为了**让每个任务尽量都能分到固定比例的CPU资源**。比如我这个任务，就固定是占用5%的CPU，不会有很大的偏差。

这种维度的调度策略，已经有比较好的实现了。那就是——Lottery Scheduling。虽然已经比较老了，但是它的思想也不是那么简单。下面就来看一看。

### 2.6.1 Basic Concept: Tickets Represent Your Share

简单举个例子，什么是彩票。假设两个进程A和B。一共有100个tiket。让A有75个，B有25个。每次（每个time slice）CPU都会选出一个中奖的tiket。谁持有它，谁就会运行。

实现起来也很简单。一个随机数，从0-99。如果结果是0-74，那么A就运行。如果是75-99，那么就是B运行。

下面是一个例子。如果CPU生成的数字，和对应的谁会执行：

```
63 85 70 39 76 17 29 41 36 39 10 99 68 83 63 62 43 0 49 12
A     A  A     A  A  A  A  A  A     A     A  A  A  A A  A
   B        B                    B     B
```

显然，这个概率是不准的。理论值B可以有25%，但是实际上只有20%。不过随着time slice越来越多，这个概率就会越来越准。 ^f23c12

### 2.6.2 Ticket Mechanisms

对于彩票调度，最重要的就是如何操作彩票。具体的机制有很多。下面开始介绍。第一种叫Ticket Currency，这个是一个彩票->货币的概念。

首先，会抽象出一个叫“用户”的概念。每个用户底下会有一堆job等待运行。每个job都有一个期望得到的彩票数量。

假设现在有两个用户A和B。其中A有两个job，B只有一个：

$$
\begin{align*}
& A
\left\{
    \begin {aligned}
         & A_1 (500)\\
         & A_2 (500)         
    \end{aligned}
\right. \\
& B \rightarrow B_1(10)
\end{align*}
$$

然后，$A_1$想要500个ticket，$A_2$想要500个ticket，而$B_2$想要10个ticket。现在，给了A和B各100个真实的ticket。

那咋办？对于A来说，~~只能~~按比例给手底下的job分，也就是平分，每个job有50个；而对于B，手底下就这一个job，所以100都给它了。

> [!attention]
> 接着[[#2.6.5 How To Assign Tickets?|读下去]]，你会发现，这里说“只能”是有问题的。其实，对于A来说，它想咋分就咋分。平分只是其中一种选择而已。而至于到底用什么方式，其实是一个很难的问题。

因此，最终的情况是：$A_1$有50个，$A_2$有50个，$B_1$有100个。最后才会用这总共200个ticket，来决定哪个job先运行。

另一个机制叫做Ticket Transfer。顾名思义，就是一个进程可以把自己的ticket转交给另一个进程。比如在CS架构下，客户端在向服务端发送请求后，为了尽快得到响应，自然希望服务端的优先级提高，因此可以将自己的ticket交给服务端，从而让它更容易被选中。在服务端处理完请求之后，也可以将ticket归还给客户端。

最后，还有一种机制叫做Ticket Inflation。虽然名字叫通货膨胀，但是操作系统里没有一般等价物，因此讨论ticket本身的价值没有意义。这里的inflation指的是，**进程自己可以选择增加或者减少自己持有ticket的数量**（而非价值）。当然，这种机制不能随便用。因为一个进程（@pdd）可以肆无忌惮地给自己增加很多ticket，让其它进程都饿死。所以，这种机制一般都用在一个“进程之间相互信任”的系统里。

### 2.6.3 Implementation

看了原著，感觉没啥好讲的，太简单了，就是一个随机数生成器，然后用这个生成数字决定谁获胜，被运行。不过，这里比较有意思的是，提了一嘴到底怎么生成，才是最随机的。这里居然还联动了一下Stack Overflow：[c - How to generate a random integer number from within a range - Stack Overflow](https://stackoverflow.com/questions/2509679/how-to-generate-a-random-integer-number-from-within-a-range)

### 2.6.4 An Example

原书103页，可以看原著，就是举了个例子，没啥。不过这里定义了一个新指标F，代表公平性。用的是两个job完成时间之比。如果两个进程几乎同时完成，那么F应该无限趋近于1。

### 2.6.5 How To Assign Tickets?

现在我们还有一个问题没说：怎么把ticket分给job？这个过程，和操作系统耦合很严重，所以方法非常多。其中一个最简单的，就是让用户自己去分配。

### 2.6.6 Stride Scheduling

上面我们介绍的所有的调度策略都有个特点：随机，我们依赖随机数来确定哪个job该运行。但是，这种随机的办法并不能准确的按比例分配，这个[[#^f23c12|之前]]我们也提到过。那么有没有一种能够准确分配时间片的调度算法，不管是什么时候都是准的呢？有的兄弟，有的。那就是Waldspurger发明的 Stride Scheduling，步幅调度。

直接说做法有点难懂，举个例子就很好办了。还是用ticket的模式。假设，ABC三个job，需要的ticket分别是：

- A 100
- B 50
- C 250

> [!tip]
> 这里想想，我的需求是什么？其实是让A分到$\dfrac{100}{100 + 50 + 250} = 25\%$的时间片；B分到$\dfrac{50}{100 + 50 + 250} = 12.5\%$的时间片；C分到$\dfrac{250}{100 + 50 + 250} = 62.5\%$的时间片。

下一步，用一个大数字处以这三个数字。为了方便计算，最好是能整除。这里我们用10000去除，就得到了每个job的stride：

- A $\dfrac{10000}{100} = 100$
- B $\dfrac{10000}{50} = 200$
- C $\dfrac{10000}{250} = 40$

这就是三个job的stride。然后，下面我们给这三个job开始计数，这个数字叫pass，从0开始。我们的规则是：

- 每次谁的pass最小，谁就能运行。如果有一样的，那选谁都行；
- **运行一个job，就给这个job的pass加上它的stride**。

这样就好理解了，下面是这三个进程的运行表格：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250125191717.png]]

- 一开始三个进程的pass都是0，所以选谁都行。我们选了A，所以A要加上100；
- 现在最小的变成了B和C，我们选B运行，然后给B加上200；
- 现在最小的就剩C了，所以它运行，给C加上40；
- 此时最小的pass还是C，所以还是C运行，所以给C加上40；
- ……

我们看这个表格里ABC的运行次数分别是2 1 5，**和ABC的ticket数之比正好相等**（$100 : 50 : 250 = 2 : 1 : 5$）。这就是步幅调度的神奇之处。

这个算法有啥缺点？我在上面走过程的时候就想到了。如果有一种情况下，一个job的pass特别小，同时stride也特别小的话，那一直都是这个job在运行。比如上面的C，就连着运行了好几次。书上举的例子也是类似：如果中间插入了一个新的job，那它的pass给设置成多少？如果设置成0的话，那岂不是说，未来一段很长的时间内，都是这个job在运行了？那肯定不太好。这就是书上所说的，Stride Scheduling是有global state的，所有job都盯着这个最小值在抢运行。但是之前的那些Lottery Scheduling就没这个问题，一直都是靠随机看命。虽然短时间内不准，但不会有这样明显的短板。

### 2.6.7 The Linux Completely Fair Scheduler (CFS)




[[Study Log/os_study/0_ostep_index|Return to Index]]