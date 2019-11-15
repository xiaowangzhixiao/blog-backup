---
title: MapReduce和Yarn简介
date: 2018-07-17 10:01:32
tags:
- 大数据
category:
- 编程
- 大数据
---

## MapReduce
一个计算框架，用于轻松编写应用程序，以可靠的容错方式在大型群集（数千个节点）的大型商业硬件上并行处理大量数据
### 设计理念
- 移动计算而不是移动数据
- 分布式计算

<!-- more -->

### 计算流程
![计算流程](/img/mapreduce.jpg)

### Mapper
- 就近计算
- 数据或计算规模较小
- 并行计算

### Reducer
- 对map阶段的输出进行汇总

### Shuffler
在mapper和reducer中间的一个步骤
把mapper的输出按照某种key值重新切分和组合成n份，把key值符合某种范围的输出送到特定的reducer那里去处理
#### 流程
![suffler](/img/shuffler.PNG)

1. map的结果存在内存中，内存满将中间结果写到磁盘里，写的过程中将每个结果进行partition分组，sort排序，spill
	1. paritition 默认为key的hashcode mod reducer的个数，目的是将计算结果分成几个reducer的输入
	2. sort 字典排序
2. merge为一个大的文件
	2. combiner 可设置，将相同key的value相加
3. fetch reducer从各个数据源取到相应分区号的数据
4. sort reducer获取到数据后排序并分组，分组是指相同key的value合并为一个列表
5. reduce程序的输入就是key带一个value list

### spilt
split的大小：
max(min.split,min(max.spilt,block))
保证一个碎片在一个block

## Yarn
![架构](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/yarn_architecture.gif)

