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
  
# 参考文献

1. [ 1986《The design of postgres》][The design of postgres]

[The design of postgres]:http://db.cs.berkeley.edu/papers/ERL-M87-06.pdf

2. [ 2016 《The Snowflake Elastic Data Warehouse》][Snowflake]

[Snowflake]:https://www.snowflake.com/wp-content/uploads/2019/06/Snowflake_SIGMOD.pdf

3. [2016 Snowflake 弹性数仓（译）][Snowflake chinese]

[Snowflake chinese]:https://zhuanlan.zhihu.com/p/357862897
