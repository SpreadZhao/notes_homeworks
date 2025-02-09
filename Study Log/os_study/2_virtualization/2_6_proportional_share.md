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

下一步，用一个大数字处以这三个数字。为了方便计算，最好是能整除。这里我们用10000去除，**就得到了每个job的stride**：

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

这个算法有啥缺点？我在上面走过程的时候就想到了。如果有一种情况下，一个job的pass特别小，同时stride也特别小的话，那一直都是这个job在运行。比如上面的C，就连着运行了好几次。书上举的例子也是类似：如果中间插入了一个新的job，那它的pass给设置成多少？如果设置成0的话，那岂不是说，未来一段很长的时间内，都是这个job在运行了？那肯定不太好。这就是书上所说的，Stride Scheduling是有global state的，所有job都盯着这个最小值在抢运行。但是之前的那些Lottery Scheduling就没这个问题，一直都是靠随机看命。虽然短时间内不准，但不会有这样明显的短板。 ^acf670

### 2.6.7 The Linux Completely Fair Scheduler (CFS)

#### 2.6.7.1 Basic Operation

我们介绍的所有的**公平**调度器，以及市面上大多数的**公平**调度器，都是基于固定的时间片的。本节的彩票调度，步幅调度就是这样。他们都是让一个程序运行固定的时间片，然后换成其它的，来实现公平调度。

下面，我们介绍linux的调度器，CFS。它不是基于固定的时间片，也就是说，我挑选一个任务该运行，但是它不会运行固定的时间。那么你就要问了：它要运行多久？你先别急，我慢慢说。

CFS的核心思想是这样的：我们之前讨论过一个问题，[[Study Log/os_study/2_virtualization/2_4_scheduling_introduction#^b0a24f|性能和公平是不可兼得的]]。在公平调度上，当然也存在这个问题：

- 如果我们越频繁选择进程去调度，那么其实公平性在提升的，因为更多任务会得到运行；但是频繁地切换进程会有额外的性能开销；
- 如果我们越**不**频繁地切换进程，那么公平性就会降低，但是性能会有提升。

CFS的核心思路，就是通过调节这个平衡，加上步幅调度的思想来完成的。

步幅调度用的是pass，对应到linux上就是virtual runtime (vruntime)。任务在运行的时候，就会积累它的vruntime。这个时间是和真实的物理时间成正比的（说实话，我感觉这句是废话，不成正比就没意义了）。也可以理解为就是真实的运行时间吧。

当需要调度时，CFS会选择vruntime最低的那个任务运行，这个和步幅调度是一样的。

然后是怎么保证平衡。第一，CFS会实时保证每个任务的运行时间。有一个参数叫`sched_latency`，一般是48ms。用它处以此时任务的数量，就能得到每个进程运行的时间片。

举个例子，ABCD 4个进程，可以得到每个进程的运行时间片是12ms。然后按照类似步幅调度的思想，增加vruntime，能得到运行情况：

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250208005215.png]]

这里后半段CD就已经结束了，只剩AB运行。这里就体现出CFS的实时性：只剩俩进程，所以每个进程的时间片从12ms变成了24ms。

> [!question] 为啥这里看着这么像round robin？
> 你可能已经注意到了，CFS里并没有ticket的概念，又或者说，每个进程的ticket都是一样的。所以这个例子看起来就像round robin一样。其实，这就是CFS本身的目标：**公平地将CPU的时间片等分给每一个参与竞争的任务**。

这里问题就来了，假设我们有100万个任务要参与竞争。那时间片会非常小，切换会太频繁。解决办法也很简单粗暴：设置一个最小的时间片，叫`min_granularity`，通常是6ms。还是刚才的例子，如果有10个任务，那么每个任务的时间片就只有4.8ms。但是它小于了`min_granularity`，所以最后还是6ms。

这样稍微损失了一些公平（因为进程数量没有等分`sched_latency`），但是也还行，同时也保证了性能。

最后稍微有个注意点，不重要，贴原文了：

> Note that CFS utilizes a periodic timer interrupt, which means it can only make decisions at fixed time intervals. This interrupt goes off frequently (e.g., every 1 ms), giving CFS a chance to wake up and determine if the current job has reached the end of its run. If a job has a time slice that is not a perfect multiple of the timer interrupt interval, that is OK; CFS tracks vruntime precisely, which means that over the long haul, it will eventually approximate ideal sharing of the CPU.

#### 2.6.7.2 Weighting (Niceness)

你的问题估计又来了：CFS既然是一个真正用在linux上的调度，那为啥只能平分CPU时间片？没有特权啥的吗？有的兄弟，有的。CFS没有使用ticket实现，而是用了一个经典的UNIX机制 —— nice level。

每一个任务可以被设置一个nice程度，值从-20到+19。数字越小，优先级越高。也就是说，你如果越优秀，那你就越不会受到调度器的注意。

每一个nice程度（prio），都会被映射成一个权重（weight）：

```c
static const int prio_to_weight[40] = {
	/* -20 */ 88761, 71755, 56483, 46273, 36291,
	/* -15 */ 29154, 23254, 18705, 14949, 11916,
	/* -10 */  9548,  7620,  6100,  4904,  3906,
	/*  -5 */  3121,  2501,  1991,  1586,  1277,
	/*   0 */  1024,   820,   655,   526,   423,
	/*   5 */   335,   272,   215,   172,   137,
	/*  10 */   110,    87,    70,    56,    45,
	/*  15 */    36,    29,    23,    18,    15
};
```

这里的意思是，如果你的nice是-20，那你的权重就是88761；如果是-19，那权重就是71755，依次类推。最后+19的权重是15。

现在还是举个例子，两个任务A和B，A比较重要，B不重要。所以给A一个低的nice，给-5；然后给B的nice是0。这样就能算出来，A的权重就是3121，B的权重是1024。那么最终两个任务分别会分走多少的时间片呢？按照下面的公式：

$$
time\_slice_k = \dfrac{weight_k}{\Sigma_{i=0}^{n-1}weight_i} \cdot sched\_latency
$$

> [!note]
> 其中$n$是任务的数量

能得到，A的slice（一次运行时间）是$\dfrac{3121}{3121+1024} \cdot sched\_latency \approx \dfrac{3}{4} \cdot sched\_latency$，B的slice大约是$\dfrac{1}{4} \cdot sched\_latency$。

这样就完了吗？我们想想，CFS是咋工作的来着：和步幅调度一样，通过增加vruntime来表示一个任务在运行，然后每次需要切换任务时，选择vruntime最低的那个任务。那现在，就AB这两个任务，我们已经知道了，A需要3/4的时间片，B需要1/4的。那么然后呢？一开始，两个任务的vruntime都是0，然后我选择A运行，然后我们运行了$\dfrac{3}{4} \cdot sched\_latency$，也就是36ms。然后我们选谁？当然选B，因为它是0。所以让B运行12ms。然后呢？运行谁？此时按照CFS的思路，应该还是运行B，因为12比36小啊。

那这个时候，我们仔细思考一下，就能发现问题：我们希望A的时间片是3/4，B的是1/4。也就是说，如果让A运行36ms，然后让B运行12ms，然后再让A运行36ms……依次类推，这样正好能满足要求。但是，现实情况却是，**B运行了12ms之后，依然还是它运行**。那这样就已经不符合要求了。

问题出在哪里？出在这个策略上。我们每次都选择vruntime最小的运行。但是，在有了权重的情况下，我们需要将权重也考虑进去。也就是说，对这个B来说啊，**虽然它只运行了12ms，但是它实际上也运行了36ms**。为啥？**因为它的权重低，所以运行时间会成比例增长**。

如何做到这一点呢？并且，在有多个任务有不同权重的情况下，也能做到**让它们的vruntime增长的速度也能契合权重呢**？其实，`prio_to_weight`这个映射里的数字之所以那么奇怪，就是考虑到了这一点。我们按照这样的公式：

$$
vruntime_i = vruntime_i + \dfrac{weight_0}{weight_i} \cdot runtime_i
$$

就能得到考虑权重后，一个任务的vruntime实际增长的速度。比如A，权重除一下是$\dfrac{1024}{3121} \approx \dfrac{1}{3}$，而B的是$\dfrac{1024}{1024} = 1$。因此，B的增长速度是A的3倍。所以在上面的例子中，B运行了12ms，**但是它的vruntime却已经增加到36ms了**。

这里注意下，分子是固定的$weight_0$，这个数字1024是设计好的，能很简单实现这样的权重计算。另外，这一坨数字也能保证，**权重相对值的无感**。举个例子，如果A的优先级是5，B的优先级是10，那么计算的结果应该和上面是一样的。因为5和10差5，-5和0也差5。至于是不是这样，自己算吧，反正它就是这么设计的。

#### 2.6.7.3 Using Red-Black Trees

这里说的是CFS是怎么安排进程的。如果你放到一个链表里，那我找到该运行的进程这个行为，本身就很耗时，因为我要一个个算到底谁的vruntime是最小的。这里用的就是红黑树去安排每一个进程。同时，也不是每一个进程都放在里面，只有在运行（或者能运行）的进程在里面，如果一个进程休眠了，或者在等IO，那就会从树里移出去放在别的地方。

![[Study Log/os_study/2_virtualization/resources/Pasted image 20250209194806.png]]

#### 2.6.7.4 Dealing With I/O And Sleeping Processes

CFS也有和[[#^acf670|步幅调度一样的问题]]。比如进程A一直在运行，B一直在休眠。等B恢复之后，那它的vruntime很小，那很长一段时间都是它运行。这样A就被饿死了。这个和权重啥的可没关系，就是B小到离谱，它weight多低，增速多快，也不可能短时间内追上其它进程的vruntime。

解决办法是，CFS会给所有刚恢复运行的任务设置一个高一点的vruntime。多高呢？一般是这棵树里最小的vruntime（注意树里只有运行任务，所以这个时间是不会很小的）。但是，这样其实也有其它的问题，就是那些频繁睡眠，但是睡眠时间很短的任务，就很难得到公平的时间片分配。

剩下的就不说了，看书吧。没啥东西了也。

[[Study Log/os_study/0_ostep_index|Return to Index]]