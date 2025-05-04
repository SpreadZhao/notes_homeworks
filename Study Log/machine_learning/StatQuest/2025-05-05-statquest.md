---
title: 2 Cross Validation
date: 2025-05-05
tags:
  - machine-learning
mtrace:
  - 2025-05-05
---

# 2 Cross Validation

我们上一章的问题先抛出来：

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250505011313.png|500]]

现在有了一堆真实的样本。那么哪些数据用来训练，哪些数据用来验证？

有一个小笨蛋提出的策略：用所有的数据做训练，然后再用所有的数据来验证！

这种做法是完全错误的。因为相当于自己当选手，自己当裁判，那肯定不行。这种情况给出的结果叫做**过度拟合**（overfit）。判断一个机器学习方法是否产生了过度拟合，就需要让它经历一些它没见过的真实数据。同时，这种错误的做法叫做Data Leakage。

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250505011802.png|500]]

一个稍微好一点的办法是：随机选一些数据用来测试，剩下的用来训练。这种做法可以避免数据泄漏，但是不知道到底选择的是不是最好的。

下面开始介绍交叉验证了。方法很简单，**分组，轮流来**。

我们把这6个点分成三组，每组两个点：

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250505012129.png|500]]

然后，先用1、2来训练，3来验证。然后再来一轮，用1验证，再来一轮，用2验证。

这样，每组都有用来被验证的机会了。上面迭代了3次，每次迭代被叫做**Fold**，所以这是一个**3-Fold Cross Validation**。

每次迭代，我们其实都可以像上一章那样，算出每次迭代的误差，然后看看到底是什么情况：

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250505012737.png|500]]

同时，我们还可以和其它的预测方法做比较，看看谁更好。比如，上一章的曲线：

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250505012919.png|500]]

还是一样的做法，得到曲线的误差，然后一起比较。这里的结果是，无论是哪一次迭代，都证明了黑色的直线误差更小。同时，因为我们用了交叉验证，所以也不用担心到底选没选对最好的测试和训练数据。

下面是一个10-Fold Cross Validation的例子，很好看懂：

![[Study Log/machine_learning/StatQuest/resources/Pasted image 20250505013218.png|500]]



