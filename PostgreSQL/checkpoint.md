checkpoint即检查点，它本质上是一种精细化管理redo_log的手段，即通过在redo_log中分段加标记的方式，减少使用redo_log回放过程中需要处理的日志量。这是checkpoint的第一性

# 如何理解checkpoint机制

建立在上面的认识上，接下看下checkpoint机制怎么来实现这件事。先抛开checkpoint机制本身不谈，假设我们自身在面临*redo_log回放时需要处理大量日志*这个问题时，我们自己可以想到的办法是什么

首先：redo_log回放只所以需要处理大量日志，是因为它不知道回放到哪里需要停下来，只能从出异常的位置一直往前回放。所以需要有个标记告诉redo_log应该停下来的位置

那么就面临以下几个问题：

1. 这个标记可以直接放在redo_log里，当回放redo_log的时候遇到，就能知道；这个标记也可以放在与redo_log无关的位置，或者单独使用一个文件
2. 这个标记应该尽量选择一个合适的时间间隔，以及一个合适的数据量间隔。因为数据库有时候会面临长时间没什么业务，有时候又会短时间内业务量爆发
3. 这个标记里面应该携带一定的信息，除了方便redo_log回放处理，还得方便真出问题的时候，能够有效的定位
4. 既然是用于redo_log回放，就不应该只考虑单机的情况，备机build、备机追赶主机也是redo_log回放的一种场景

> 个人理解主要应该解决的是以上4个问题，其实除了这些，checkpoint机制还考虑了更多的问题，比如是不是应该做成手动命令对用户提供、是不是可以趁机回收部分redo_log等。这种我们先抛开不看，因为这对我们理解checkpoint的核心机制或者需要解决的核心问题并无影响

首先我们可以回答问题1，这个问题最为简单，答案是一目了然的：标记应该直接放在redo_log中。原因有几点：

1. redo_log这个机制本身就是为了记录对数据库系统有修改或者有影响的操作，而checkpoint其实也是其中一种。复用redo_log自身的机制是理所当然的，不然就是redo_log机制不称职（或者在redo_log之外搞出了其它特殊化的东西，这从设计上来说就很不好，而且不便于后续的扩展和维护）
2. redo_log已有的机制也方便checkpoint的使用，复用已有流程减轻工作量
3. PG的主备流复制采用的是redo_log（即问题4），如果弄出一个与redo_log无关的标记，反而增加流复制的处理逻辑

解决了问题1，下面我们再详细结合PG的源码来看下，PG是如何解决剩下的几个问题的

# checkpoint实现

## 不只是一个标记

checkpoint的初衷虽然是作为一个redo_log的标记，但是并非真的只靠一个标记信息就能满足要求。因为它需要解决的问题是：**当redo_log写下这个标记时，必须确保所有数据都已经落盘**

要实现这个目的，首先可以看看已有的BgWriter和WalWriter等流程能不能办到，但是显然不太现实。因为BgWriter只负责持久化堆表这类脏页，而WalWriter只负责持久化日志的脏页。所以需要改造已有的流程，**让checkpoint在执行的时候，既能够持久化当前的日志，也要能持久化当前的堆表、索引等脏页**

抛开数据库中经常要考虑的加锁、异常判断等等情况不谈，当前的checkpoint核心流程如下：

> 此处，不去详细展开更多的细节，比如checkpoint还分`XLOG_CHECKPOINT_SHUTDOWN`和`XLOG_CHECKPOINT_ONLINE`等等，让我们关注主干。这些细节功能都是基于可用性、可靠性而产生的一些修修补补的工作

```c
CreateCheckPoint
     ├── CheckPointGuts  // 完成除redo_log外的所有内存脏数据落盘
     │
     ├── XLogInsert(RM_XLOG_ID, shutdown ? XLOG_CHECKPOINT_SHUTDOWN : XLOG_CHECKPOINT_ONLINE);
     │
     └── XLogFlush // 写有checkpoint记录的redo_log最后落盘
```

然后我们来看一下redo_log中的一个checkpoint标记长什么样。其结构如下。无需关注除了第一个参数`XLogRecPtr redo`之外的其它参数：

> redo_log中的记录需要遵循它自己的一套规则，具体可以参考[redo_log](./redo_log.md)中的详细介绍，此处不做展开

1. 对于一个checkpoint标记而言，它自身只需要`XLogRecPtr redo`这一条信息就够了(就是这条信息，也是防并发考虑的，可参见源码`xlog.c`中`CreateCheckPoint`在初始化`XLogRecPtr redo`参数时的注释)，达成这样它已经完成了一个标记的本质功能，即能够让redo_log正确的识别它
2. 其它参数都是出于checkpoint参与redo_log回放、以及怎么让数据库系统快速的恢复到异常前的状态而加入的。这部分参数可能随着PG版本的迭代和特性的更替有所变化。这部分也将放在[怎么参与回放](#怎么参与回放)中详细展开

```c
typedef struct CheckPoint
{
	XLogRecPtr	redo;			/* checkpoint之后的下一条redo_log记录的位置 */
	TimeLineID	ThisTimeLineID; /* current TLI */
	TimeLineID	PrevTimeLineID; /* previous TLI, if this record begins a new
								 * timeline (equals ThisTimeLineID otherwise) */
	bool		fullPageWrites; /* current full_page_writes */
	FullTransactionId nextXid;	/* next free transaction ID */
	Oid			nextOid;		/* next free OID */
	MultiXactId nextMulti;		/* next free MultiXactId */
	MultiXactOffset nextMultiOffset;	/* next free MultiXact offset */
	TransactionId oldestXid;	/* cluster-wide minimum datfrozenxid */
	Oid			oldestXidDB;	/* database with minimum datfrozenxid */
	MultiXactId oldestMulti;	/* cluster-wide minimum datminmxid */
	Oid			oldestMultiDB;	/* database with minimum datminmxid */
	pg_time_t	time;			/* time stamp of checkpoint */
	TransactionId oldestCommitTsXid;	/* oldest Xid with valid commit
										 * timestamp */
	TransactionId newestCommitTsXid;	/* newest Xid with valid commit
										 * timestamp */

	/*
	 * Oldest XID still running. This is only needed to initialize hot standby
	 * mode from an online checkpoint, so we only bother calculating this for
	 * online checkpoints and only when wal_level is replica. Otherwise it's
	 * set to InvalidTransactionId.
	 */
	TransactionId oldestActiveXid;
} CheckPoint;
```

## 需要一个常驻进程

此处是为了解决问题2，即：`这个标记应该尽量选择一个合适的时间间隔，以及一个合适的数据量间隔。因为数据库有时候会面临长时间没什么业务，有时候又会短时间内业务量爆发`

其实这个问题比较好解决，常见的做法就是为此生成一个定时任务，以此确保一定的时间间隔会执行一次checkpoint并生成标记。而这个定时任务与`合适的数据量间隔`触发条件共用一套核心流程即可

而添加一个定时任务就需要考虑什么时候拉起、什么时候关闭以及异常处理等等。另外，作为一个任务，通常是需要开放一些配置参数，从而在不同实际场景下适配修改的。这部分详细内容就不再赘述了，直接参考[background_process相关小节](./background_process.md#checkpointer)即可

## 怎么参与回放

此处是为了解决问题3和问题4，即：`这个标记里面应该携带一定的信息，除了方便redo_log回放处理，还得方便真出问题的时候，能够有效的定位`和`既然是用于redo_log回放，就不应该只考虑单机的情况，备机build、备机追赶主机也是redo_log回放的一种场景`

对于问题3，这里其实就是前面`CheckPoint`结构体中，除了`XLogRecPtr redo`之外的其它参数所需要解决的。而对于问题4，主备机除开某些特殊场景不太一样之外，它们本质上都会走同一套redo_log回放流程。此处不针对这些细枝末节的特殊场景展开，只是单纯看下checkpoint在回放中的作用，详细的redo_log回放请参考[how_to_redo](./how_to_redo.md)中的介绍

