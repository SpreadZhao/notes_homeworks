![[Lecture Notes/Software Architecture/resources/Pasted image 20230502131848.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502131911.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502131957.png]]

# 1. Data Flow

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502134157.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502134215.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502135014.png]]

> These three topologies above, which one is not suitable for data flow architecture?
> 
> The answer is: A. Cause there's no way to **ensure** the dependency tree of datas in each component.

## 1.1 Batch Sequential

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502135541.png]]

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502135651.png]]

**The next system's input must be the <u>whole result</u> of the previous system.**

![[Lecture Notes/Software Architecture/resources/Pasted image 20230502135943.png]]

## 1.2 Pipe and Filter
