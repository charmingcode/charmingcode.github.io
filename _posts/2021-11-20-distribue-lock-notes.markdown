---
layout: post
title:  "distribute lock - notes"
date:   2021-11-20 10:23:24 +0800
categories: jekyll update
---

# distributed lock

* 分布式锁，可以用来协调，比如leader election，但是用作互斥是不安全的。比如lock timeout的时候，如何保证你在做的需要加锁的action 是在lock active time内的，很难保证因为存在和其他远程服务的网络的timeout。
所以应用层必须提供一个fencing 机制（或者一个token），否则仅分布式锁是不安全的;
比如一个dfs，可以提供SealFile接口（即此file readonly）；然后应用系统设计上，要通过文件来保证目录的一致性或原子性即可。另外有的系统提供了目录/文件创建等操作的token的乐观机制也可以保证操作的安全性。
  * 文件禁写 Seal File
  * 目录禁写 inline File token 目录内的rename createfile 禁掉

# 参考文献

1. [ 1986《chubby》][Chubby]

[Chubby]:https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/chubby-osdi06.pdf?spm=ata.21736010.0.0.1e65334c5fm4XT&file=chubby-osdi06.pdf

2. [ 2016 《Is RedLock Safe?》][RedLock]

[RedLock]:https://www.snowflake.com/wp-content/uploads/2019/06/Snowflake_SIGMOD.pdf

3. [《How to do distributed locking ?》][martin]

[martin]:https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html?spm=ata.21736010.0.0.1e65334c5fm4XT

4. [ 关于一致性协议, 分布式锁以及如何使用分布式锁 ][talk]
   
[talk]:https://zhuanlan.zhihu.com/p/31666418?from_voters_page=true
