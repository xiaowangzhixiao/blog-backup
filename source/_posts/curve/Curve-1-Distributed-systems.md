---
title: curve学习之分布式系统基础知识
toc: true
comments: true
p: curve/
date: 2020-12-30 12:25:56
tags:
- 分布式系统
category:
- 编程
- curve
---

最近参与了网易的c计划，简单讲就是给网易的开源存储项目做点小贡献，再深入的学习一下分布式系统、存储和rpc等技术，从今天开始准备每天拿出两个小时学习curve，加油！！！

今天首先根据[文章](http://book.mixu.net/distsys/index.html)学习一下分布式系统的基本概念，以下基本内容摘自该文章第一章，基础知识。

## curve简介
Curve是网易开发的一套开源的分布式存储系统，官网[链接](https://opencurve.github.io/)，开源[地址](https://github.com/opencurve/curve)

官网介绍
> CURVE是网易自主设计研发的高性能、高可用、高可靠分布式存储系统，具有非常良好的扩展性。基于该存储底座可以打造适用于不同应用场景的存储系统，如块存储、对象存储、云原生数据库等。CURVE 的设计开发始终围绕三个理念：一是顺应当前存储硬件设施发展趋势，做到软硬件结合打造顶级的存储产品；二是秉持 “Simple Can be harder than complex”，了解问题本质情况下选择最简单的方案解决问题；三是拥抱开源，在充分调研的前提下使用优秀的开源项目组件，避免造轮子。
>
>当前我们基于CURVE已经实现了高性能块存储系统，支持快照克隆和恢复 ,支持QEMU虚拟机和物理机NBD设备两种挂载方式, 在网易内部作为高性能云盘使用。

## 分布式系统概念


> 分布式系统就是处理以下两个分布式的结果的影响
> - 信息以光速传播
> - 独立的部分之间失败是相互独立的

### 基本概念
#### 什么是分布式编程
> Distributed programming is the art of solving the same problem that you can solve on a single computer using multiple computers.
分布式编程就是在多台计算机上解决可以在一台计算机上解决的相同问题的艺术

通常，计算机要解决的有两个问题
1. 存储
2. 计算

当单台计算机无法完成计算或者存储时，通常来说换硬件可以解决，例如赛博朋克2077现在的电脑带不起来，那就换3090显卡呀...哈哈哈

单机硬件升级总是有极限的，更何况有各种成本限制，所以大家为了提升性能，打上了多机分布式的算盘。

<!-- more -->

### 指标

我们使用分布式的多台计算机去解决问题，首先想要获得的提升就是可伸缩性，当我们处理的问题规模变大时，各种性能指标不能变差。
#### Scalability
> is the ability of a system, network, or process, to handle a growing amount of work in a capable manner or its ability to be enlarged to accommodate that growth.

- 大小可伸缩性：添加更多节点应使系统线性更快;增加数据集不应增加延迟
- 地理可扩展性：应该可以使用多个数据中心来缩短响应用户查询的时间，同时以某种合理的方式处理跨数据中心延迟。
- 管理可伸缩性：添加更多节点不应增加系统的管理成本（例如，管理员与计算机的比率）。

#### Performance
> is characterized by the amount of useful work accomplished by a computer system compared to the time and resources used.
性能是系统完成的有用工作量比使用的时间和资源

与其相关的有一下几个指标
- 响应时间/时延
- 吞吐量
- 计算资源利用率

##### Latency
Latent 文章中用Latency的词根来解释latency，表示的是数据到达之后一直到生效的时间间隔，潜伏期，很有意思的解释

#### Availability
> the proportion of time a system is in a functioning condition. If a user cannot access the system, it is said to be unavailable.

系统处于正常运行状态的时间比例。如果用户无法访问系统，则说它不可用。
Availability = uptime / (uptime + downtime)

使用分布式系统的另一个目标就是获得可用性，在一系列不可靠的组件之上构建一个可靠的系统，基本操作就是容错

##### Fault tolerance
> ability of a system to behave in a well-defined manner once faults occur
容错可以归结为：定义预期中的故障，然后设计一个系统或一种能够容忍故障的算法。你不能容忍你没有考虑过的错误。

### 目标达成的约束
分布式系统受两个物理因素的限制：

- 节点数（随着所需的存储和计算容量的增加）
- 节点之间的距离（信息以光速传播）

在这些约束下工作：

- 独立节点数量的增加会增加系统中发生故障的可能性（降低可用性并增加管理成本）
- 独立节点数量的增加可能会增加节点之间的通信需求（随着规模增加而降低性能）
- 地理距离的增加增加了远程节点之间通信的最小延迟（降低某些操作的性能）


### 基本技术
- 分区
- 复制

分布式系统可以将数据拆分为多个节点（分区），以允许进行更多并行处理。也可以在不同节点上复制或缓存它，以缩短客户端和服务器之间的距离，并提高容错能力（复制）。

![20201230220251](https://cdn.jsdelivr.net/gh/xiaowangzhixiao/pic@master/blogs/20201230220251.png)

#### Partitioning
分区是将数据集划分为较小的独立集合;这用于减少数据集增长的影响，因为每个分区都是数据的子集。
- 分区通过限制要检查的数据量和在同一分区中定位相关数据来提高性能
- 分区通过允许分区独立失败来提高可用性，增加在牺牲可用性之前需要失败的节点数
#### Replication
复制在多台计算机上复制相同的数据，允许更多的服务器参与计算。
- 复制通过使额外的计算能力和带宽适用于数据的新副本来提高性能
- 复制通过创建数据的其他副本来提高可用性，增加在牺牲可用性之前需要失败的节点数

复制遵循一致性模型