# MySQL Performace

## 架构

![](http://assets.processon.com/chart_image/5c049cd2e4b0c1c52df5b8fe.png)

### 连接管理与安全性

服务器维护

### 优化与执查询缓存行

- 解析 SQL 语句成解析树，并对其优化
- explain 查询优化相关信息
- 对于 select，先查询缓存

## 并发控制

- 读写锁
  - 读锁 = 共享锁，可以共享
  - 写锁 = 排它锁，不共享，阻塞读锁
- 锁粒度
  - 表锁：锁住一个表，锁开销小，但并发效率不高
  - 行级锁：锁住一行，锁开销大，当并发效率高，在存储引擎层实现
  
## 事务

- 特性：ACID
  - 原子性，Atomicity，要么全部执行，要么都不执行
  - 一致性，Consistency，总是从一个一致性状态，到另一个一致性状态
  - 隔离性，Isolation，一个事务对其他事务是不可见的
  - 持久性，Durability，永久保存到数据库中
  
## 隔离级别

- READ UNCOMMITTED，未提交读，可以读到其他事务没有提交的修改（脏读，Dirty Read）
- READ COMMITTED，提交读，只能读到其他事务提交的，在同一个事务中有可能读到不同的数据（其他事务提交的数据，不可重复读）
- REPEATABLE READ，可重复读，在同一个事务中，重复读到的已经存在数据都是相同的，但如果查询范围数据，新增的数据也会被读出来（幻读，Phantom Read），MySQL 通过 MVCC（多版本并发控制，Multiversion Concurrency Control） 解决
- SERIALIZABLE，可串行化，事务一个一个执行，效率低下

## 死锁

- 同时操作资源
- 死锁检测、超时机制
- MySQL 将持有最少行级排它锁的事务进行回滚

## 事务日志

- 修改时，修改内存拷贝，再将其修改行为记录到事务日志，数据慢慢刷回磁盘
- 如果崩溃，可以通过事务日志修复

## MySQL 事务

- 自动提交，show variables like 'autocommit'; set autocommit = 1;
- 设置隔离级别，set session transaction isolation level read committed;

## MVCC

- 多版本并发控制，Multiversion Concurrency Control
- 每行保存隐藏的值
  - 行的创建时间
  - 行的过期时间（删除时间）
- 保存的值是系统版本号，没开始一个新的事务，系统版本号增加
- CRUD
  - select，只查找版本早于当前事务版本的数据行
  - insert，保存当前系统版本号作为创建时间
  - delete，保存当前系统版本号作为删除时间
  - update，insert 新 + delete 老
- 保证了大多数形况不用加锁，但需要更多的维护空间

## 表

- show table status like 'user';
- [返回结果值](https://www.cnblogs.com/lxwphp/p/8109261.html)
  - Row_Format，Dynamic 行长度可变，Fixed 固定，Compressed 压缩
  - Collation，字符集，字符排序规则

## InnoDB

- 数据存储在表空间
- MVCC 控制高并发，四个隔离级别
- 聚簇（cu，不是 zu，😂）引擎，第一级索引，第二级索引保留了叶子节点，值是主键

## 转换引擎

- alter table mytable engine=innodb
- mysqldump 

## 安装

[详情](https://www.cnblogs.com/xiaopotian/p/8196464.html)

## 基准测试

[sysbench](https://github.com/akopytov/sysbench)
在 /etc/my.cnf 增加 default_authentication_plugin=mysql_native_password

### 命令

- CPU: sysbench --test=cpu --cpu-max-prime=20000 run
- IO: 
  - sysbench --test=fileio --file-total-size=1G --file-test-mode=rndrw --max-time=300 --max-requests=0 prepare
  - sysbench --test=fileio --file-total-size=1G --file-test-mode=rndrw --max-time=300 --max-requests=0 run
- OLTP(Online Transaction Process):
  - create database test(in MySQL)
  - sysbench /usr/share/sysbench/oltp_read_write.lua --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-db=test --mysql-user=root --table_size=1000000 --tables=1 --threads=8 --events=5000 --report-interval=10 --time=0 prepare
  - sysbench /usr/share/sysbench/oltp_read_write.lua --mysql-host=127.0.0.1 --mysql-port=3306 --mysql-db=test --mysql-user=root --table_size=1000000 --tables=1 --threads=8 --events=5000 --report-interval=10 --time=0 run

### 结果

CPU
```
Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Prime numbers limit: 20000

Initializing worker threads...

Threads started!

CPU speed:
    events per second:   353.91

General statistics:
    total time:                          10.0002s
    total number of events:              3540

Latency (ms):
         min:                                    2.37
         avg:                                    2.82
         max:                                    5.22
         95th percentile:                        3.49
         sum:                                 9991.08

Threads fairness:
    events (avg/stddev):           3540.0000/0.00
    execution time (avg/stddev):   9.9911/0.00
```

IO
```
Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Extra file open flags: (none)
128 files, 8MiB each
1GiB total file size
Block size 16KiB
Number of IO requests: 0
Read/Write ratio for combined random IO test: 1.50
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random r/w test
Initializing worker threads...

Threads started!

File operations:
    reads/s:                      10375.65
    writes/s:                     6917.10
    fsyncs/s:                     22134.96

Throughput:
    read, MiB/s:                  162.12
    written, MiB/s:               108.08

General statistics:
    total time:                          300.0401s
    total number of events:              11829826

Latency (ms):
         min:                                    0.00
         avg:                                    0.02
         max:                                  762.15
         95th percentile:                        0.09
         sum:                               291641.79

Threads fairness:
    events (avg/stddev):           11829826.0000/0.00
    execution time (avg/stddev):   291.6418/0.00

```

OLTP
```
Running the test with following options:
Number of threads: 8
Report intermediate results every 10 second(s)
Initializing random number generator from current time


Initializing worker threads...

Threads started!

[ 10s ] thds: 8 tps: 492.07 qps: 9853.57 (r/w/o: 6898.83/1969.89/984.85) lat (ms,95%): 28.67 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            70000
        write:                           20000
        other:                           10000
        total:                           100000
    transactions:                        5000   (490.88 per sec.)
    queries:                             100000 (9817.55 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          10.1842s
    total number of events:              5000

Latency (ms):
         min:                                    4.90
         avg:                                   16.27
         max:                                   90.16
         95th percentile:                       28.67
         sum:                                81359.15

Threads fairness:
    events (avg/stddev):           625.0000/13.28
    execution time (avg/stddev):   10.1699/0.00
```
