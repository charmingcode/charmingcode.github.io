---
layout: post
title:  "Leveldb 源码分析 - snapshot"
date:   2021-05-05 21:22:24 +0800
categories: jekyll update
---

#### API

{% highlight c++ %}
const Snapshot* DBImpl::GetSnapshot() {
   MutexLock l(&mutex_);
   return snapshots_.New(versions_->LastSequence());
}
 
void DBImpl::ReleaseSnapshot(const Snapshot* s) {
   MutexLock l(&mutex_);
   snapshots_.Delete(reinterpret_cast<const SnapshotImpl*>(s));
}
{% endhighlight %}


行级别的snapshot / compact；  gc策略要耦合 smallest_snapshot seq 的 ; 
缺点就是仅分了两个组，一个是 <= smallest_snapshot_seq 一个是> smallest_snapshot_seq 数据集；

另外由于snapshot_seq的存在，导致compact中的delete marker 不能轻易删除了，只能判断大level无此数据才能delete.

这里面如果要设计多版本的话，多版本中的delete marker 即使当前的compact 中有当前row key的更大version的非delete row，此时的delete marker,也不能purge。

重要的delete 逻辑 以及delete的marker purge策略；
非delete marker , 仅保留在 smallest_snapshot中的最大version( max_versions = 1)
delte marker （要判断这个是当前smallest_snapshot中的最大version的row，且保证high level 无data，即后面没有比这个小的version 的row 需要此row delete marker gc的数据。）


文件级别的 snapshot / compact 
缺点：如果snapshot 过多会存在大量文件无法进行compact

思考：
另外目前的leveldb 实现的snapshot 是 没有持久化的。
有了snapshot 全局的计算逻辑/比如preAgg/rollup/delete-marker/data gc 都要考虑对snapshot的影响。