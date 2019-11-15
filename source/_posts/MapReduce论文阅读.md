---
title: MapReduce论文阅读
toc: true
comments: true
date: 2019-07-08 23:46:23
tags: 
- MapReduce
- 论文阅读
category: 
- 编程
- MIT课程学习
- 6.824
---

## 摘要

MapReduce是一个编程模型，是处理和生成大数据集的相关实现。用户实现一个map函数用来处理一个key/value对生成一组key/value中间结果，一个reduce函数根据相同的中间结果的key合并所有的中间结果。许多现实世界里的任务都可以在这个任务中表现出来，例如在这篇论文中提到的。

<!-- more -->

## 介绍

大部分计算涉及到用一个map操作对输入的每条记录处理得到一系列中间结果kv对，并且使用一个reduce操作对所有相同key的value进行处理，适当的结合导出数据。使用带有用户指定的map和reduce操作的函数模型，使我们能够轻松的对大型计算进行并行化，并使用重新执行作为容错的主要机制。

## 编程模型

Map由用户编写，接受一个输入对并生成一组中间键/值对。MapReduce库将所有有相同的中间键I关联的中间值组合在一起，并将他们传递给Reduce函数。

Reduce函数由用户编写，它接收一个中间键I和该键的一组值。它将这些值合并在一起，形成一个可能更小的值集，中间值通过一个迭代器提供给用户的reduce函数，这允许我们处理太大而无法装入内存的值列表。

![函数形式](/img/MapReduce.png)

## 例子

- word count
    - map函数切分单词，并将单词w作为key，1作为value发射出去
    - reduce函数将所有value求和，求出key的个数
    - 伪代码
```
    map(String key, String value):
        // key: document name 
        // value: document contents for each word w in value: 
        EmitIntermediate(w, "1");

    reduce(String key, Iterator values):
        // key: a word 
        // values: a list of counts int result = 0; 
        for each v in values: 
            result += ParseInt(v); 
        Emit(AsString(result));
```

- 分布式Grep：如果map函数与提供的模式匹配，则发出一行；reduce恒等输出
- URL访问频率囧叔：map函数处理web页面请求的日志并输出URL；reduce将相同URL的值相加，并发出一个URL，total count对
- 反向web链接图：map输出target、source对，表示在名为source的页面中找到的目标url的每个链接；reduce函数链接与给定目标URL关联的所有URL列表，并发出一对target，list(source)
- 反向索引：map解析每个文档，并发出一系列word、document ID对；reduce函数接收给定单词的所有对，对相应的文档ID进行排序，发出word、list(doc ID)对
- 分布性排序：map函数从记录中提取键，发出key，record对；reduce函数恒等输出，这个依赖mapreduce库中的排序属性。

## 执行
![执行概述](/img/mapreduceExec.png)

## 容错
### worker
主控定期向每个工人发出ping的信号，如果一定时间按内没有收到来自worker的响应，则判断为失败，重新调度任务。
### master
定时写检查点，如果master终止，可以从最后一个检查点状态启动一个新的副本。
