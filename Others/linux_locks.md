介绍Linux系统中的信号量和锁

| 类别            | 适用范围      | 原理                                                         | 接口                                                         | 说明                                                         |
| --------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 互斥信号量      | 用户态/线程间 | 同一时间只有一个线程能够占用该锁，其它尝试获取该锁的线程将会陷入睡眠 | pthread_mutexattr_init     pthread_mutexattr_setprotocol     pthread_mutex_lock     pthread_mutex_unlock | 1.该锁只能用于互斥，锁只能由拥有者释放          2.可以指定锁的类型     PTHREAD_MUTEX_NORMAL：同一线程重复获取锁将会死锁     PTHREAD_MUTEX_ERRORCHECK：同一线程重复获取锁将会报错     PTHREAD_MUTEX_RECURSIVE：同一线程重复获取锁将会增加锁计数，需要unlock同样的次数才会真正解锁          3.可以指定互斥锁的属性     PTHREAD_PRIO_INHERIT：优先级继承。拥有该锁的线程将以被该锁阻塞的高优先级的线程的优先级运行。     PTHREAD_PRIO_PROTECT：优先级保护。线程将以所有拥有锁的优先级上限中的最高优先级运行。          4.获取锁的方式为FIFO |
| 计数信号量      | 用户态/线程间 | 即  posix无名（内存）信号量                                  | sem_init  （初始化）     sem_wait （p）     sem_post （v）   | 1.资源数可以大于1          2.获取和释放者可以不一致          3.posix无名信号量还可用于进程间通信，但必须是父子进程 |
| 二进制信号量    | 用户态/线程间 | 即  资源数为1的特殊posix无名信号量                           | sem_init  （初始化）     sem_wait （p）     sem_post （v）   | 1.获取和释放者可以不一致          2.posix无名信号量还可用于进程间通信，但必须是父子进程 |
| posix有名信号量 | 用户态/进程间 | 原理是共享内存，sem_open会在/dev/shm/下创建sem.xxx的文件。   | sem_open  （创建，需要指定名字）     sem_wait （p）     sem_post （v）     sem_close （关闭）     sem_unlink （销毁） | 1.open了必须close，用完后必须销毁，否则会资源泄露。     2.获取和释放者可以不一致 |
| System V信号量  | 用户态/进程间 | System  V IPC机制使用了系统范围类的资源，ipc key系统唯一     | ftok  （获取时ipc key）     semget （获取信号量）     semctl （设置初始值）     semop （p/v操作） | 其它System  V IPC机制还包括：shmget（共享内存） msgget（消息队列） |
| 文件锁          | 用户态/进程间 | 即对文件进行加锁。文件锁分为三种：flock、lockf和fcntl        | int  fcntl (int fd, int cmd, struct flock *lock)           int flock(int fd, int operation)          int lockf(int fd, int cmd, off_t len); | 【fcntl】     1.cmd取值：F_GETLK（获取锁信息）、F_SETLK（加解锁，加锁失败则返回）、F_SETLKW（加解锁，加锁失败则睡眠）     2.lock控制是加读锁（F_RDLCK）、加写锁（F_WRLCK）还是解锁（F_UNLCK）     通过该函数还可以控制加锁的位置     3.lock可以控制加锁的位置（SEEK_SET、 SEEK_CUR、SEEK_END以及偏移）     4.另外，fcntl还可以用于租约锁（A对文件加了租约锁，如果B操作文件，A会收到通知）          【flock】     1.operation取值LOCK_SH（共享锁）、LOCK_EX（排他锁）、LOCK_UN（解锁）     2.如果加锁请求不能被立即满足，那么系统调用 flock() 会阻塞当前进程          【lockf】     1.lockf基于fcntl实现     2.cmd取值：F_ULOCK（解锁）、F_LOCK（加锁，如果不成功则阻塞）、F_TLOCK（加锁，如果不成功则返回）、F_TEST（测试是否存在锁）     3.len表示从文件中当前偏移量开始的锁定长度 |
| 关调度          | 内核态        | 即禁止CPU进行调度                                            | preempt_disable     preempt_enable                           | 关抢占以CPU的核为单位，不是操作整个CPU                       |
| 关中断          | 内核态        | 不响应中断                                                   | local_irq_disable     local_irq_enable     local_bh_disable     local_bh_enable | 这里只是屏蔽中断，并不是让中断不产生（这需要控制gic等）      |
| 内核自旋锁      | 内核态        | 获取锁时会先关闭CPU抢占，获取不到锁时会一直自旋（死等），不会陷入睡眠。 | spin_lock     spin_unlock                                    | 1.还存在关中断版本     关闭硬中断：spin_lock_irq、spin_lock_irqsave     关闭软中断：spin_lock_bh |
| 内核读写锁      | 内核态        | 读操作时，写操作必须等待；写操作时，读操作也需要的等待。读锁（共享锁）可以多个进程同时持有；写锁（排它锁）只有一个进程能持有。 | read_lock     read_unlock     write_lock     write_unlock    | 还存在关中断的版本：     read_lock_irq     read_lock_bh     write_lock_irq     write_lock_bh |
| 内核信号量      | 内核态        | 获取不到信号量时会陷入睡眠                                   | DECLARE_MUTEX     down （p）     up （v）                    | down的时候可以选择睡眠的类型：     down_interruptible     down_killable     down_trylock     down_timeout |
| 内核读写信号量  | 内核态        | 获取不到信号量时会陷入睡眠                                   | down_read/up_read     down_write/up_write                    |                                                              |
| 顺序锁          | 内核态        | 1.读写锁的优化版本     2.读写操作之间不再互斥，写的优先级高于读，即检测到正在写的时候，读操作暂停（cpu_relax）32.写操作之间依旧互斥     3.顺序锁保存有一个计数，读操作前后的计数如果不一致，那么读操作会重试 | write_seqlock     write_sequnlock     read_seqbegin     read_seqretry | 一般用于读操作很多、写操作很少的场景                         |
| RCU锁           | 内核态        | 1.RCU（Read-Copy  Update）锁是对读写锁的一种优化     2.读不需要任何锁。在写时首先拷贝一个副本，然后对副本进行修改，最后使用一个回调在适当的时机把指向原来数据的指针重新指向新的被修改的数据。 | rcu_read_lock     rcu_read_unlock     rcu_dereference     rcu_assign_pointer | 对于读多写少的场景，机制的开销比较小，性能会大幅度提升，但是如果写操作较多时，开销将会增大，性能不一定会有所提升 |

# 参考资料

1. [linux内核原子操作的实现](https://blog.csdn.net/vividonly/article/details/6599502)
2. [Linux内核29-原子操作](https://tupelo-shen.github.io/2020/04/08/Linux%E5%86%85%E6%A0%B829-%E5%8E%9F%E5%AD%90%E6%93%8D%E4%BD%9C/)

