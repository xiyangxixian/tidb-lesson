# 测试目的

用不同工具测试 tidb 集群的性能，并找到性能瓶颈所在。

# 部署环境

## 部署机器

4台相同规格的物理机，操作系统 centos 7.3，内核 3.10，cpu 64核，内存128G，磁盘 8* 480G SSD(RAID 50)，2*万兆网卡

|   项目   | 配置                                                         | 台数 |
| :------: | :----------------------------------------------------------- | :--: |
|   TiDB   | 操作系统 centos 7.3，内核 3.10，cpu 64核，内存128G，磁盘 8* 480G SSD(RAID 50)，2*万兆网卡 |  1   |
| TiKV& PD | 操作系统 centos 7.3，内核 3.10，cpu 64核，内存128G，磁盘 8* 480G SSD(RAID 50)，2*万兆网卡 |  3   |

## 集群拓扑

| 节点名称 | 部署服务           |
| -------- | ------------------ |
| Node1    | tidb:4000          |
| Node2    | Pd:2379,tikv:20160 |
| Node3    | Pd:2379,tikv:20160 |
| Node4    | Pd:2379,tikv:20160 |

## 集群配置

### tidb 关键参数

```
[performance]
max-procs = 0
```

### tikv 关键参数

```
[readpool]
[readpool.coprocessor]
# high-concurrency 默认 64 * 0.8 ≈ 50
# normal-concurrency 默认 64 * 0.8 ≈ 50
# low-concurrency 默认 64 * 0.8 ≈ 50
[storage]
[storage.block-cache]
# 默认系统内存总大小 45%， 128 * 0.45 ≈ 57
```

# 集群测试

## sysbench 测试

### sysbench 配置

```
mysql-host=10.xxx.xxx.xxx
mysql-port=4000
mysql-user=sysbench
mysql-password=******
mysql-db=test
time=60
report-interval=10
db-driver=mysql
```

### 准备数据

```
sysbench --config-file=config oltp_point_select --tables=16 --table-size=500000 prepare
```

### point select

命令：

```
sysbench --threads=8 --config-file=config oltp_point_select --tables=16 --table-size=500000 run
```

输出：

```
SQL statistics:                                                                                                          [0/1387]
    queries performed:
        read:                            1466950
        write:                           0
        other:                           0
        total:                           1466950
    transactions:                        1466950 (24447.77 per sec.)
    queries:                             1466950 (24447.77 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.0017s
    total number of events:              1466950

Latency (ms):
         min:                                    0.20
         avg:                                    0.33
         max:                                   38.91
         95th percentile:                        0.39
         sum:                               479232.97

Threads fairness:
    events (avg/stddev):           183368.7500/66.14
    execution time (avg/stddev):   59.9041/0.00
```

命令：

```
sysbench --threads=64 --config-file=config oltp_point_select --tables=16 --table-size=500000 run
```

输出：

```
SQL statistics:
    queries performed:
        read:                            7070584
        write:                           0
        other:                           0
        total:                           7070584
    transactions:                        7070584 (117818.05 per sec.)
    queries:                             7070584 (117818.05 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.0110s
    total number of events:              7070584

Latency (ms):
         min:                                    0.21
         avg:                                    0.54
         max:                                   35.25
         95th percentile:                        0.80
         sum:                              3836598.11

Threads fairness:
    events (avg/stddev):           110477.8750/147.74
    execution time (avg/stddev):   59.9468/0.00
```

### update index 

命令：

```
sysbench --threads=8 --config-file=config oltp_update_index --tables=16 --table-size=500000 run
```

输出：

```
SQL statistics:
    queries performed:
        read:                            0
        write:                           138745
        other:                           0
        total:                           138745
    transactions:                        138745 (2312.10 per sec.)
    queries:                             138745 (2312.10 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.0065s
    total number of events:              138745

Latency (ms):
         min:                                    1.87
         avg:                                    3.46
         max:                                  205.18
         95th percentile:                        6.21
         sum:                               479958.68

Threads fairness:
    events (avg/stddev):           17343.1250/39.08
    execution time (avg/stddev):   59.9948/0.00
```

命令：

```
sysbench --threads=64 --config-file=config oltp_update_index --tables=16 --table-size=500000 run
```

输出：

```
SQL statistics:
    queries performed:
        read:                            0
        write:                           548066
        other:                           0
        total:                           548066
    transactions:                        548066 (9126.12 per sec.)
    queries:                             548066 (9126.12 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.0529s
    total number of events:              548066

Latency (ms):
         min:                                    1.82
         avg:                                    7.01
         max:                                 3178.79
         95th percentile:                       11.87
         sum:                              3840097.18

Threads fairness:
    events (avg/stddev):           8563.5312/159.75
    execution time (avg/stddev):   60.0015/0.01
```

### read only

命令：

```
sysbench --threads=8 --config-file=config oltp_read_only --tables=16 --table-size=500000 run
```

输出：

```
SQL statistics:
    queries performed:
        read:                            599088
        write:                           0
        other:                           85584
        total:                           684672
    transactions:                        42792  (713.06 per sec.)
    queries:                             684672 (11409.01 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.0098s
    total number of events:              42792

Latency (ms):
         min:                                    7.17
         avg:                                   11.22
         max:                                   51.81
         95th percentile:                       19.65
         sum:                               479955.12

Threads fairness:
    events (avg/stddev):           5349.0000/9.23
    execution time (avg/stddev):   59.9944/0.00
```

命令：

```
sysbench --threads=64 --config-file=config oltp_read_only --tables=16 --table-size=500000 run
```

输出：

```
SQL statistics:
    queries performed:
        read:                            2417632
        write:                           0
        other:                           345376
        total:                           2763008
    transactions:                        172688 (2877.19 per sec.)
    queries:                             2763008 (46034.98 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.0180s
    total number of events:              172688

Latency (ms):
         min:                                    9.16
         avg:                                   22.24
         max:                                  174.12
         95th percentile:                       30.81
         sum:                              3840197.62

Threads fairness:
    events (avg/stddev):           2698.2500/7.24
    execution time (avg/stddev):   60.0031/0.00
```



### write only

命令：

```
sysbench --threads=8 --config-file=config oltp_write_only --tables=16 --table-size=500000 run
```

输出：

```
SQL statistics:
    queries performed:
        read:                            0
        write:                           244748
        other:                           122374
        total:                           367122
    transactions:                        61187  (1019.65 per sec.)
    queries:                             367122 (6117.91 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.0060s
    total number of events:              61187

Latency (ms):
         min:                                    4.15
         avg:                                    7.84
         max:                                   48.25
         95th percentile:                       12.30
         sum:                               479927.86

Threads fairness:
    events (avg/stddev):           7648.3750/9.67
    execution time (avg/stddev):   59.9910/0.00
```

命令：

```
sysbench --threads=64 --config-file=config oltp_write_only --tables=16 --table-size=500000 run
```

输出：

```
SQL statistics:
    queries performed:
        read:                            0
        write:                           896852
        other:                           448426
        total:                           1345278
    transactions:                        224213 (3735.65 per sec.)
    queries:                             1345278 (22413.90 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.0181s
    total number of events:              224213

Latency (ms):
         min:                                    4.91
         avg:                                   17.13
         max:                                  233.63
         95th percentile:                       26.20
         sum:                              3840126.85

Threads fairness:
    events (avg/stddev):           3503.3281/9.91
    execution time (avg/stddev):   60.0020/0.00
```

### read write

命令：

```
sysbench --threads=8 --config-file=config oltp_read_write --tables=16 --table-size=500000 run
```

输出：

```
SQL statistics:
    queries performed:
        read:                            342748
        write:                           97928
        other:                           48964
        total:                           489640
    transactions:                        24482  (407.88 per sec.)
    queries:                             489640 (8157.66 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.0203s
    total number of events:              24482

Latency (ms):
         min:                                   12.39
         avg:                                   19.61
         max:                                  164.78
         95th percentile:                       29.72
         sum:                               480041.41

Threads fairness:
    events (avg/stddev):           3060.2500/4.49
    execution time (avg/stddev):   60.0052/0.00
```

命令：

```
sysbench --threads=64 --config-file=config oltp_read_write --tables=16 --table-size=500000 run
```

输出：

```
SQL statistics:
    queries performed:
        read:                            1647982
        write:                           470852
        other:                           235426
        total:                           2354260
    transactions:                        117713 (1960.69 per sec.)
    queries:                             2354260 (39213.76 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.0348s
    total number of events:              117713

Latency (ms):
         min:                                   17.36
         avg:                                   32.63
         max:                                   99.07
         95th percentile:                       44.17
         sum:                              3840822.65

Threads fairness:
    events (avg/stddev):           1839.2656/5.43
    execution time (avg/stddev):   60.0129/0.01
```



## go-tpc

命令：

```
./bin/go-ycsb load mysql -P workloads/workloada -p recordcount=100000 -p mysql.host=10.xxx.xxx.xxx
./bin/go-ycsb run mysql -P workloads/workloada -p recordcount=100000 -p mysql.host=10.xxx.xxx.xxx
```

输出：

```
READ   - Takes(s): 1.6, Count: 488, OPS: 310.8, Avg(us): 835, Min(us): 641, Max(us): 4465, 99th(us): 2000, 99.9th(us): 5000, 99.99th(us): 5000
UPDATE - Takes(s): 1.6, Count: 512, OPS: 325.7, Avg(us): 2276, Min(us): 1956, Max(us): 8643, 99th(us): 4000, 99.9th(us): 9000, 99.99th(us): 9000
```

命令：

```
./bin/go-ycsb load mysql -P workloads/workloadb -p recordcount=100000 -p mysql.host=10.xxx.xxx.xxx
./bin/go-ycsb run mysql -P workloads/workloadb -p recordcount=100000 -p mysql.host=10.xxx.xxx.xxx
```

输出：

```

```

命令：

```
./bin/go-ycsb load mysql -P workloads/workloadc -p recordcount=500000 -p mysql.host=10.xxx.xxx.xxx
```

输出：

```

```

命令：

```
./bin/go-ycsb load mysql -P workloads/workloadd -p recordcount=500000 -p mysql.host=10.xxx.xxx.xxx
```

输出：

```

```

命令：

```
./bin/go-ycsb load mysql -P workloads/workloade -p recordcount=500000 -p mysql.host=10.xxx.xxx.xxx
```

输出：

```

```

命令：

```
./bin/go-ycsb load mysql -P workloads/workloadf -p recordcount=500000 -p mysql.host=10.xxx.xxx.xxx
```

输出：

```

```

