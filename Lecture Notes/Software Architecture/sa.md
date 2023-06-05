# 1. Architecture Style

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502131848.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502131911.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502131957.png]]

* 组件
* 连接件
* 它们之间的关系

## 1.1 Data Flow

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502134157.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502134215.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502135014.png]]

> These three topologies above, which one is not suitable for data flow architecture?
> 
> The answer is: A. Cause there's no way to **ensure** the dependency tree of datas in each component.

### 1.1.1 Batch Sequential

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502135541.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502135651.png]]

**The next system's input must be the <u>whole result</u> of the previous system.**

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502135943.png]]

### 1.1.2 Pipe and Filter

**If only the data comes, components can work**, which is the biggest difference than the Batch Sequential. Such feature enables **parallel** among components.

![[Lecture Notes/Software Architecture/resources/Pasted image 20230503112524.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230503112601.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230503112646.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230503112850.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230503112902.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230503113010.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230503113135.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230503113151.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230503113338.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230503113441.png]]

## 1.2 Call Return

### 1.2.1 Main Program and Subroutines

其实就是C语言的面向过程设计。**和面向对象不同，无法实现数据的封装和隐藏**。根据功能划分，将大问题分解成若干子问题。每个子问题由一个或多个子程序来完成，并让**主程序来调用这些子程序**。这种特性导致了主程序一定是知道子程序的执行过程的，也就没法实现封装。并且，如果系统或者问题过于复杂，如果只用这种方法的话，子程序和子子程序之间的交互就会过于复杂。

* Component: 主程序，子过程
* Connector: 主程序怎么调用子过程，显示的
* Topology: 根据功能划分出来的结构

优点：可以设计大型程序；缺点：过于复杂不行，可靠性有问题。这种风格通常会和其他风格一起来定义。

### 1.2.2 OOP

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605132437.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605133030.png]]

优点：

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605133524.png]]

缺点：

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605134023.png]]

### 1.2.3 Layered

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605134217.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605134229.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605134242.png]]

## 1.3 Data-centered

### 1.3.1 Repository

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605140936.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605141042.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605141406.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605141426.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230605141731.png]]

### 1.3.2 Blackboard

