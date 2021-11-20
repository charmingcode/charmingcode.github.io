---
layout: post
title:  "paper reading - notes[WIP]"
date:   2021-11-20 21:52:24 +0800
categories: jekyll update
---

# DataBase

## PostgreSql

### 1986《The degisn of postgres》
* 提出了对象关系数据库
* 支持复杂对象
* 方便拓展，以及优化器识别
* 采用多进程架构，当时多线程支持各个操作系统还不成熟

## SnowFlake

### 2016《The Snowflake Elastic Data Warehouse》
* 2016发布的论文，2020年上市，目前看到云原生数据库最早的论文，比阿里的PolarDB要早，但确实是更high level的，以及提出主要设计理念和产品理念
* 存储计算分离架构 = 云存储 + 弹性计算
* virtual warehouse
  * 弹性与隔离
  * 本地缓存与文件级的stealing (clickhouse 是做到block级别的stealing)
  * 执行引擎 = 列存+向量化+push模型（提到了用的是push模型，减少控制逻辑，提高cache效率，能有多少呢？）
* cloud services
  * 查询管理与优化: DQL/DML时候收集统计信息，执行的时候延迟决策执行计划，也提出了一点无索引设计考虑，可以降低QO 搜索空间的复杂度。
  * 并发控制:实现了SI， 快照隔离
  * 剪枝：做了很多Meta统计，进行数据剪枝，进一步讨论没有采用B+索引，是避免数据膨胀
* 重点功能
  * 半结构化/无schema 数据查询：通过一系列优化能做到同传统schema数据做到性能接近（列存，乐观转换等）。

## PolarDB

### 2017 云栖大会
* 会议中提出了PolarFs
* 低延迟流式Replication 
  * 优化1 : 只复制WAL meta
  * 优化2 : 页面回放优化（follower仅按需设置内存的中page为outdate，也可以理解为不需要回放不在mem的page） 
  * 优化3 : DDL锁优化，避免主进行drop table,而follower节点的drop table卡住，导致leader 无法进行wal sender。  
* 讨论
  * replication 设计： share-storage的架构，自带的replication友好，但是需要管理好leader节点的资源GC逻辑(比如table drop, file remove等)； B+ tree的无wal的存储结构设计可以采用上面的优化2，但是lsm则不能按需回放，因为lsm的wal的数据都是要先写入memtable进行对外可读的，不像B+tree 内存数据仅为cache，实际的数据都直接write on disk了； 另外上面的优化3，是share_storage模式核心要解决的问题，本质就是资源共享了，那么GC机制就要好好设计了，比如replica 跟leader的增量数据通过肯定是基于wal的，能作为进度表达的就是sequence，那么follower 共享的资源，就要都能通过sequence来进行关联，即follower未sync到的sequence 可能被共享的资源要保证不能被删除，比如data file，而sync到的file，可以通过dfs的hard link，将生命周期管理，从leader wal的sequnce管理，交接给follower自己,而dfs file hard link就很好的将leader/follower的file owner进行了解耦。
  
### 2018 《PolarFS- An Ultra-low Latency and Failure Resilient Distributed File System for Shared Storage Cloud Database》
* 软硬件技术，将dfs的latency 做到逼近local fs
* RDMA 
* SPDK
* ParallelRaft (这里面的提到的优化，可以对比下kudu的raft实现)

### 2019 《Cloud-Native Database Systems at Alibaba- Opportunities and Challenges》
* 主要简介了阿里的数据库产品
* PolarDB
* PolarDB-X
* ADB
* SDDP (自动化运维)

# 参考文献

1. [ 1986《The design of postgres》][The design of postgres]

[The design of postgres]:http://db.cs.berkeley.edu/papers/ERL-M87-06.pdf

2. [ 2016 《The Snowflake Elastic Data Warehouse》][Snowflake]

[Snowflake]:https://www.snowflake.com/wp-content/uploads/2019/06/Snowflake_SIGMOD.pdf

3. [2016 Snowflake 弹性数仓（译）][Snowflake chinese]

[Snowflake chinese]:https://zhuanlan.zhihu.com/p/357862897
