# MySQL Performace

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
