# Database

## Mysql

Mysql中的一些总结

1. - [ ] [undo](./Database/Mysql/undo.md)：Mysql中的undo机制介绍

## PostgreSQL

PostgreSQL的一些总结

1. - [ ] [background_process](./Database/PostgreSQL/background_process.md)：简单介绍了Linux进程、线程结构，以及PG中的几个后台常驻进程
2. - [ ] [memory_structure](./Database/PostgreSQL/memory_structure.md)：介绍PG中的内存结构，如共享内存、本地存库，堆表和日志的内存
3. - [ ] [swap_algo_of_memory](./Database/PostgreSQL/swap_algo_of_memory.md)：介绍PG中不同内存的换入换出算法
4. - [ ] [logs_in_pg](./Database/PostgreSQL/logs_in_pg.md)：介绍PG中的各种日志
   - - [ ] [redo_log](./Database/PostgreSQL/redo_log.md)
   - - [ ] [clog](./Database/PostgreSQL/clog.md)
5. - [ ] [locks_in_pg](./Database/PostgreSQL/locks_in_pg.md)：介绍操作系统中典型的几种锁结构，以及PG中使用到的锁
6. - [ ] [MVCC](./Database/PostgreSQL/MVCC.md)：介绍PG的MVCC实现
7. - [ ] [vacuum](./Database/PostgreSQL/vacuum,md)：PG中的回收机制
8. - [x] [checkpoint](./Database/PostgreSQL/checkpoint.md)：简单介绍PG中的检查点
   - - [ ] [deep_in_checkpoint](./Database/PostgreSQL/deep_in_checkpoint.md) : 更进一步的深入介绍checkpoint
10. - [ ] [how_to_redo](./Database/PostgreSQL/how_to_redo.md)：PG如何回放日志

## Distributed PostgreSQL

以PostgreSQL为原型开发的分布式数据库系统（如Greenplum、PGXL等）的一些总结

1. - [ ] [data_distribution](./Database/Distributed_PostgreSQL/data_distribution.md)：介绍多节点上的数据分布方式
2. - [ ] [redistribution](./Database/Distributed_PostgreSQL/redistribution.md)：介绍多节点的数据重分布，即扩缩容
3. - [ ] [election_of_datanode](./Database/Distributed_PostgreSQL/election_of_datanode.md)：数据节点的选主机制

## OpenGauss

1. - [ ] [parellel_redo](./Database/OpenGauss/parellel_redo.md)：介绍并行回放的机制



## Comparison

主要归纳不同数据库的特点和比较，故放在`Database`的根路径下

1. - [ ] [compare_of_pg_and_mysql](./Database/compare_of_pg_and_mysql.md)：比较Database/PostgreSQL和Mysql

## Common

主要归纳一些数据库系统的共性问题，故放在`Database`的根路径下

1. - [ ] [Isolation_level](./Database/Isolation_level.md): 事务的隔离级别

# Others

讨论一些与数据库有关的周边问题

1. - [ ] [protocol_of_transaction_consistency](./Others/protocol_of_transaction_consistency.md)：介绍几种事务一致性协议
