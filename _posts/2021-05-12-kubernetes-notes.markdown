---
layout: post
title:  "kubernetes - notes[WIP]"
date:   2021-05-12 10:08:24 +0800
categories: jekyll update
---

# 前言

kubernetes 是希腊文，意思是`舵手`，带领我们安全地到达未知水域。首次接触基本了解是一个容器编排系统，但是有人称其为`数据中心操作系统`，不解，故来进一步探索未知水域。

目前了解到的k8s诞生的背景，得益于微服务理念的兴起，由`垂直扩容`改进为`水平扩容`，当系统整体负载上升，需要弹性扩容的时候，如果是单体应用，那么扩容的成本大概率很大，因为可能仅仅是某个局部的处理逻辑产生了资源瓶颈需要扩容，比如磁盘、内存或者CPU,此时如果将单体系统，按照功能合理的拆分，在用户侧能直观的受益点就是，扩容成本可以下降，比如给耗CPU更多的处理微服务扩容计算机型或者给占用更多磁盘空间的微服务扩展存储机型； 从研发视角，架构更合理，每个微服务可以各自迭代。

# 重要概念

## POD

为何需要pod关于为何需要pod这种容器？为何不直接使用容器？为何甚至需要同时运行多个容器？难道不能简单地把所有进程都放在一个单独的容器中吗？

接下来我们将一一回答上述问题。为何多个容器比单个容器中包含多个进程要好想象一个由多个进程组成的应用程序，无论是通过ipc（进程间通信）还是本地存储文件进行通信，都要求它们运行于同一台机器仁。在Kubemetes中，我们经常在容器中运行进程，由于每一个容器都非常像一台独立的机器，此时你可能认为在单个容器中运行多个进程是合乎逻辑的，然而在实践中这种做法并不合理。<font color='red'>容器被设计为每个容器只运行一个进程（除非进程本身产生子进程）。如果在单个容器中运行多个不相关的进程，那么保持所有进程运行、管理它们的日志等将会是我们的责任。</font>例如，我们需要包含一种在进程崩溃时能够自动重启的机制。同时这些进程都将记录到相同的标准输出中，而此时我们将很难确定每个进程分别记录了什么。综上所述，我们需要让每个进程运行于自己的容器中，而这就是<font color='red'>Docker和Kubernetes期望使用的方式</font>。

pod 也是`扩缩容的基本单位`。

对于在生产中运行的pod，一定要定义一个存活探针`livenessProbe`。没有探针的话，Kubernetes无法知道你的应用是否还活着。只要进程还在运行，Kubernetes会认为容器是健康。同样每个pod也可能有自己的owner，owner级联的GC策略，见文件2。

<br/>
## Volume


<br/>
## Service

<br/>
## ReplicaSet 
了解ReplicaController，但是使用ReplcaSet 而不是ReplicaController


<br/>
## Deployment


<br/>
## StatefulSet

<br/>
## CRD

### schema

### event

### envtest

# 实践

![k8s_in_action_00]({{ "https://github.com/charmingcode/charmingcode.github.io/blob/master/_site/assets/imgs/k8s_in_action_00.png?raw=true" }})

# 参考文献

1. [Kubernetes in action 中文版][Kubernetes-in-action]
2. [Owners and dependents][Owners-and-dependents]
3. [Kubebuilder][Kubebuilder-link]

[Kubernetes-in-action]: http://product.dangdang.com/26439199.html
[Owners-and-dependents]:https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/
[Kubebuilder-link]:https://book.kubebuilder.io/introduction.html
