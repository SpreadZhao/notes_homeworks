---
title: 1 Fundamental Concepts in Machine Learning
date: 2025-04-23
tags:
  - machine-learning
mtrace:
  - 2025-04-23
---

# 1 Fundamental Concepts in Machine Learning

## 1.1 Two Silly Examples

先举两个 Silly Example！

*Do you like StatQuest*?

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250423001517.png]]

- 你喜欢Silly Songs吗（Silly Songs是开头唱的一个挺尬的歌）？
	- 如果你喜欢Silly Songs，那你喜欢机器学习吗？
		- 如果你喜欢机器学习，那你喜欢StatQuest！
		- 如果你不喜欢机器学习，那你喜欢统计学吗？
			- 如果你喜欢统计学，那你喜欢StatQuest！
			- 如果你不喜欢统计学，那你可能不太喜欢StatQuest :(
	- 如果你不喜欢SillySons，那你喜欢机器学习吗？
	- 。。。（懒得打了）

上面的例子叫一个决策树（Decision Tree）。这棵树的作用是，**预测**（predict）一个人到底喜不喜欢StatQuest。

换句话说，这棵树可以将一个人**归类**（classify）成喜欢StatQuest和不喜欢的。

**决策树就是一种机器学习**。这棵树在推导人到底喜不喜欢StatQuest过程，就是机器学习的过程！

第二个例子：

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250423002329.png|400]]

横轴是一个人吃多少yam（芋头好像是，反正就是碳水。。），纵轴是一个人能跑多快。

- 像我这样的菜鸡，吃不了多少，也跑不了多快，所以在最左下角。
- 一些体格肥硕，缺乏运动的人，可能吃的很多，但是跑的也不会很快。
- 。。。
- 博尔特就在最右上角，他吃的多，跑得快。

这里可以发现，我们可以在中间画一条线做预测：

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250423002531.png]]

总之，机器学习就两件事：

1. Predictions
2. Classifications

## 1.2 Basic Concepts in Machine Learning

下面是一些机器学习的基本概念：

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250423002753.png]]

这些红色的点，是我们真实获取的数据，它们被用来训练，叫做Training Data。

中间黑色的线，是我们用来预测的一种方式。

不难发现，看起来，我们可以画一个绿色的曲线，它可能比这条直线更加符合真实情况：

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250423003113.png]]

但是，光猜不行，我们得证明。

证明的方式：**我们再找几个人**！

这几个人就叫做测试数据（Testing Data）。

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250423003929.png|400]]

接下来呢？我们一个个看。首先是第一个人：

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250423004031.png|400]]

现在标出来的，就是我们预测的Speed和实际的Speed的差值。我们将这个差值记下来。之后对于第2,3,4个人，都一样。最后把这些差值加起来。

接下来对于曲线也一样，最终得到结果：

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250423004241.png|200]]

看！绿色的曲线的差值比黑色的直线的差值还大。证明虽然看起来很美好，但是实际上不中用。

最终，我们选择了黑色的线做预测。

这个例子中，我们总结出两个事情：

1. 我们使用Testing Data来评估（evaluate）机器学习方法（黑色直线，绿色曲线）；
2. 有些机器学习的方法看似很匹配真实的Training Data对吧，但是不要被迷惑了。

> [!note]
> 和训练数据匹配的挺好，但是一做预测就拉胯。这种情况叫做偏差-方差均衡（[Bias–variance tradeoff](https://en.wikipedia.org/wiki/Bias%E2%80%93variance_tradeoff)）

此时，我们其实已经介绍了一个非常炫酷的名词，叫做深度学习卷积神经网络。但是实际上，这些概念都是围绕着怎么做预测，怎么测试，让它预测的更准服务的。具体来说，就是把Testing Data给玩出花来。

现在，回到一开始我们的那个决策树。这种类型的方法怎么测试？还是一样，找测试数据。直接找几个人，让他们回答那几个问题，最后问他们喜不喜欢StatQuest。

然后，还是一个人一个人来，等走到最后一步，对比真实情况和我们预测的结果。总之，我们找的人越多，最后校准的结果就应该是越准的。

最后，还有一个问题：这些数据，哪些用作Training Data，哪些用作Testing Data？答案是，有方法确定。当然方法是什么？下一章揭晓。

> [!attention]
> 下面的内容从视频切换到了书，所以内容会有点不同，但是不影响介绍概念。

接下来是机器学习的一些概念。我们一个一个说。

- Independent and Dependent Variables

什么是变量？我们刚刚统计的横轴和纵轴就是变量。在上面的例子中，吃的东西的数量和速度就是变量。而对于最一开始的例子，我们其实能画成一个表格：每一列就是，是否喜欢机器学习、喜欢xxx？所以，这些是否喜欢xxx也是变量。

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250505010129.png|500]]

然后，变量有什么不同么？还是上面的例子，纵轴的数据是我们**要预测**的，所以它的变化其实依赖于横轴的变量。所以它叫做**Dependent Variable**；相反，横轴的变量不是我们要预测的，所以它就是一个**Independent Feature**，或者叫**Feature**（特征）。

> [!comment]
> 感觉就是函数里的因变量和自变量。

在上面的例子中，我们只用了横轴一个变量（视频里是吃的芋头数量，书上是体重）来预测纵轴的值。但实际上，可以有许多个特征来一起预测最终的值。

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250505010746.png|500]]

---

- Discrete and Continuous Data

离散数据和连续数据。这个其实没啥好说的，上面的表格里，体重就是一个连续的数据，而喜欢的颜色，只有几个取值，所以是离散的数据。