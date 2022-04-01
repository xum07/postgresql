MVCC，全称Multi Version Concurrency Control，即并发版本控制。它本质是为了解决并发情况下，数据的ACID问题(Atomicity Consistency Isolation Durability)中ACI部分

MVCC有自己的一套理论和算法，这里不做详细展开，如有兴趣可以自行阅读：
1. [Multiversion Concurrency Control - Theory and Algorithms](https://www.researchgate.net/publication/220225682_Multiversion_Concurrency_Control_-_Theory_and_Algorithms)
2. [Multi Versions and the Performance of Optimistic Concurrency Control](https://ftp.cs.wisc.edu/pub/techreports/1983/TR517.pdf)

为了方便理解，个人觉得MVCC其实可以看做两个问题：
1. MV(Multi Version)：数据可以有多个版本。实现多版本就涉及到数据的组织形式、怎么在多个版本中选择、怎么清理等问题
2. CC(Concurrency Control)：对数据的并发访问。通常就是怎么做并发控制

> MC和CC两个其实相互关联、相互影响的，它们之间的关系就如同数据结构与算法。在解决MVCC问题的时候，必然需要设计一种合理的结构，再实现一种基于该结构的算法。
  此处只是为了便于理解，从不同的侧重点来看待它们解决的问题
  
