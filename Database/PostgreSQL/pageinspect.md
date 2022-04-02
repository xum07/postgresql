在使用`pageinspect`前，先确保PostgreSQL已经正确按照并运行

1. 使用前可以先行确认下`pageinspect`是否已经安装，执行`select * from pg_extension`查看已安装的扩展

   > 下图中已经存在`extname`为`pageinspect`的插件，表明已经安装过了。若是未安装，则该数据不存在

   ```sql
   test=# select * from pg_extension;
     oid  |   extname   | extowner | extnamespace | extrelocatable | extversion | extconfig | extcondition 
   -------+-------------+----------+--------------+----------------+------------+-----------+--------------
    13572 | plpgsql     |       10 |           11 | f              | 1.0        |           | 
    24633 | pageinspect |       10 |         2200 | t              | 1.9        |           | 
   (2 rows)
   ```

2. 进入`pageinspect`源码目录，`$PGHOME/contrib/pageinspect`，直接执行`make install`即可，安装路径自动继承PostgreSQL的配置，无需额外设置

3. 使用`psql`连接到数据库，执行`create extension pageinspect`进行安装即可，对应打印如下

   > 需注意：`pageinspect`所在的系统表`pg_extension`不是全局共享的，也就是每个database下都有一份。如有需要，必须在所需的database下自行执行安装

   ```sql
   test=# create extension pageinspect;
   CREATE EXTENSIO
   ```

4. 后续使用即可。类似如下：

   ```sql
   test=# select * from page_header(get_raw_page('student','main',0));
       lsn    | checksum | flags | lower | upper | special | pagesize | version | prune_xid 
   -----------+----------+-------+-------+-------+---------+----------+---------+-----------
    0/1771F88 |        0 |     5 |    48 |  7984 |    8192 |     8192 |       4 |         0
   (1 row)
   ```

