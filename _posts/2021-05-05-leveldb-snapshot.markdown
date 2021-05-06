---
layout: post
title:  "Leveldb 源码分析 - snapshot"
date:   2021-05-05 21:22:24 +0800
categories: jekyll update
---

### API

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

### SnapShot 使用者视角

1. 不能指定时间点，仅支持当前的时间点的snapshot构建，因为历史时间存在compact
2. 某个snapshot对应的时间点的存量数据读取
   - 数据比较
   - 存量数据的索引构建
3. 存量数据依然可以TTL （看设计的snapshot语义）
4. leveldb的实现不支持snapshot的持久化

### SnapShot 设计

Snapshot的设计核心在于compact中对于GC策略的影响。

{% highlight c++ %}
if (snapshots_.empty()) {
   compact->smallest_snapshot = versions_->LastSequence();
} else {
   compact->smallest_snapshot = snapshots_.oldest()->sequence_number();
}

Iterator* input = versions_->MakeInputIterator(compact->compaction);
SequenceNumber last_sequence_for_key = kMaxSequenceNumber;
 // Handle key/value, add to state, etc.
    bool drop = false;
    if (!ParseInternalKey(key, &ikey)) {
      // Do not hide error keys
      current_user_key.clear();
      has_current_user_key = false;
      last_sequence_for_key = kMaxSequenceNumber;
    } else {
      if (!has_current_user_key ||
          user_comparator()->Compare(ikey.user_key,
                                     Slice(current_user_key)) != 0) {
        // First occurrence of this user key
        current_user_key.assign(ikey.user_key.data(), ikey.user_key.size());
        has_current_user_key = true;
        last_sequence_for_key = kMaxSequenceNumber;
      }

      // 在非多version的系统下，同一个row key的不同version，
      // 在一个snapshot内仅保留当前snapshot内最大version即可。
      if (last_sequence_for_key <= compact->smallest_snapshot) {
        // Hidden by an newer entry for same user key
        drop = true;    // (A)
      } else if (ikey.type == kTypeDeletion &&
                 ikey.sequence <= compact->smallest_snapshot &&
                 compact->compaction->IsBaseLevelForKey(ikey.user_key)) {
        // kTypeDeletion的userkey的purge策略，只有保证其他未参与的sstfile，
        // 不再需要此delete marker 才可以purge。
        // For this user key:
        // (1) there is no data in higher levels
        // (2) data in lower levels will have larger sequence numbers
        // (3) data in layers that are being compacted here and have
        //     smaller sequence numbers will be dropped in the next
        //     few iterations of this loop (by rule (A) above).
        // Therefore this deletion marker is obsolete and can be dropped.
        drop = true;
      }

      last_sequence_for_key = ikey.sequence;
    }
{% endhighlight %}

重要的delete 逻辑 以及delete的marker purge策略；
* 非delete marker , 仅保留在 smallest_snapshot中的最大version( max_versions = 1)
* delete marker （要判断这个是当前smallest_snapshot中的最大version的row，且保证high level 无data，即后面没有比这个小的version 的row 需要此row delete marker gc的数据。）

### 讨论
有了snapshot 全局的计算逻辑/比如preAgg/rollup/delete-marker/data gc 都要考虑对snapshot的影响。

* 行级别的snapshot 
   - 优点：内存空间占用少，一个snapshot仅记录一个seq即可，且不依赖mem-flush/compact pic，但是依赖seq;
   - 缺点：空间放大较大（如果想GC的更有效，GC策略复杂度大，所有的snapshot下的同一个userkey的最大version都要保留）
      仅分了两个组，一个是 <= smallest_snapshot_seq 一个是> smallest_snapshot_seq 数据集；且需要针对单user_key的gc策略产生影响——由于snapshot_seq的存在，导致compact中的delete marker 不能轻易删除了，只能判断大level无此数据才能delete.

* 文件级别的 snapshot / compact 
create snapshot时候 flush mem，一个snapshot为file list, 且持久化到manifest中
   - 优点：不依赖seq，因为在compact pick时候已经分组，所以不影响compact的user-key gc逻辑就能较彻底的GC;因为snapshot引入的空间放大小。
   - 缺点：影响compact pick, 如果snapshot 过多会存在大量文件无法进行compact，导致文件数过多


