# Mysql

Mysql中的一些总结

1. [undo](./Mysql/undo.md)：Mysql中的undo机制介绍

# PostgreSQL

PostgreSQL的一些总结

1. [background_process](./PostgreSQL/background_process.md)：简单介绍了Linux进程、线程结构，以及PG中的几个后台常驻进程
2. [memory_structure](./PostgreSQL/memory_structure.md)：介绍PG中的内存结构，如共享内存、本地存库，堆表和日志的内存
3. [swap_algo_of_memory](./PostgreSQL/swap_algo_of_memory.md)：介绍PG中不同内存的换入换出算法
4. [logs_in_pg](./PostgreSQL/logs_in_pg.md)：介绍PG中的各种日志
   - [redo_log](./PostgreSQL/redo_log.md)
   - [clog](./PostgreSQL/clog.md)
5. [locks_in_pg](./PostgreSQL/locks_in_pg.md)：介绍操作系统中典型的几种锁结构，以及PG中使用到的锁
6. [MVCC](./PostgreSQL/MVCC.md)：介绍PG的MVCC实现
7. [vacuum](./PostgreSQL/vacuum,md)：PG中的回收机制
8. [checkpoint](./PostgreSQL/checkpoint.md)：PG中的检查点
9. [how_to_redo](./PostgreSQL/how_to_redo.md)：PG如何回放日志

# Distributed PostgreSQL

以PostgreSQL为原型开发的分布式数据库系统（如Greenplum、PGXL等）的一些总结

1. [data_distribution](./Distributed_PostgreSQL/data_distribution.md)：介绍多节点上的数据分布方式
2. [redistribution](./Distributed_PostgreSQL/redistribution.md)：介绍多节点的数据重分布，即扩缩容
3. [election_of_datanode](./Distributed_PostgreSQL/election_of_datanode.md)：数据节点的选主机制

# Others

主要归纳不同数据库的特点和比较，放在根路径下

1. [compare_of_pg_and_mysql](./compare_of_pg_and_mysql.md)：比较PostgreSQL和Mysql