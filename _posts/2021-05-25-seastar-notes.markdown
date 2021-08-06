---
layout: post
title:  "seastar - notes[WIP]"
date:   2021-05-25 10:43:24 +0800
categories: jekyll update
---

# 前言

## Asynchronous programming

share data vs share nothing
* 减少负面影响 cache line bouncing and memory fences

异步服务编程遇到的两大挑战
* 复杂性：很长的回调链
* 非阻塞：依然存在阻塞函数需要引入多线程，比如基于disk io的读写实现，比如mmap
此外需要考虑
* 现代机器：多core和多层级的memory (L1 cache -> memory) ,  悲观思想-不可扩展的lock，乐观思想-共享内存/无锁机制（原子操作/内存屏障）但是对性能影响也是不能忽略，尤其core的扩展也是有限制的 
* 编程语言：高级语言，比如java, js 以及类似的现代语言是方便使用的，但是每个语言的设计都有与上面列出的问题进行解决。但是这些语言考虑了可移植，所以给开发者关键的代码少了一些控制权利。而对于真正的性能优化，我们需要一个编程语言可以提供开发者完全的控制，以及无运行时损耗的语言。

基于上面的挑战进而提出了`seastar`。


## Seastar

Seastar是一个应用框架，它几乎将操作系统所提供的抽象完整地搬移到了用户态中，以减少操作系统的抽象开销，实现软硬件一体化。[Scylla] [Scylla] 基于Seasatr开发——一个类似Apache Cassandra的高性能NoSql database。

`The [obj = ...] capture syntax we used here is new to C++14. This is the main reason why Seastar requires C++14, and does not support older C++11 compilers.`

# 重要特性

## Share nothing


<br/>

## Future and Promise(fpc)

### ready_future
如果一个 future 在当下就已经有结果了，不必等到未来某个时刻， 我们把这个 future 称为 ready_future。

ready_future 的不同之处在于:

```
1. ready_future 可以单独使用，不必关联一个promise；
2. ready_future.then(lambda) 会把传入的lambda立即执行掉， 也就是说这个lambda没有机会放入任务队列；
3. 而not_ready_future每次执行，产生的新任务都会被放入任务队列，然后依此取出来执行。
```

<br/>

### do_with

{% highlight c++ %}
seastar::future<> f() {
    T obj; // wrong! will be destroyed too soon!
    return slow_op(obj);
}
{% endhighlight %}

`do_with` saves the given object on the heap, and calls the given lambda with a reference to the new object. Finally it ensures that the new object is destroyed after the returned future is resolved. Usually, do_with is given an rvalue, i.e., an unnamed temporary object or an std::move()ed object, and do_with moves that object into its final place on the heap. do_with returns a future which resolves after everything described above is done (the lambda’s future is resolved and the object is destroyed).

{% highlight c++ %}
seastar::future<> f() {
    return seastar::do_with(T(), [] (auto& obj) {
        // obj is passed by reference to slow_op, and this is fine:
        return slow_op(obj);
    }
}

{% endhighlight %}


## Message passing

<br/>

## High-performance networking 


# 参考文献

1. [ScyllaDB][Scylla]
2. [用户态操作系统之一 Seastar简介][Seastar]
3. [Asynchronous Programming with Seastar][AsynProgramWithSeastar]

[Scylla]:http://www.scylladb.com/
[SeaStar]:https://zhuanlan.zhihu.com/p/38771059
[AsynProgramWithSeastar]:http://docs.seastar.io/master/split/index.html

