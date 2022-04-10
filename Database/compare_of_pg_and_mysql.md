# 区别点

| 机制      | PpostgreSQL | Mysql  |
| --------- | ----------- | ------ |
| 线程/进程 | 多进程      | 多线程 |

## 线程/进程

在PG中各个后台任务采用的是进程管理的模式，而Mysql采用的是多线程管理的模式。它们各有优缺点，但总体而言，当前主流的数据库都更青睐于多线程（Gauss200曾经花大代价将原来继承自PG的多进程改造为多线程）

> 必须事先说明的是：在Linux系统中，进程和线程其实没有本质差别，可以参见[background_process](./PostgreSQL/background_process.md)；但是在windows系统上，进程和线程在本质实现上就有很大差别，线程创建、管理很方便，而进程创建的代价就比较大

# 参考资料

1. [Double Write简介](http://3ms.huawei.com/hi/group/2191/wiki_5855018.html)

