---
title: redis实战1——简介
toc: true
comments: true
date: 2020-01-26 19:27:55
tags:
- 读书笔记
- redis实战
category:
- 编程
- redis
---

## 简介
Redis是一个存储Key-Value数据的内存数据库，支持时间点转储和日志追加两种方式的数据持久化，支持基于主从复制的双机热备。

## 数据结构简介
Redis存储Key-Value类型的数据，Key只支持字符串，Value支持5种数据类型，分别是String、List、Set、Hash和ZSet。

| 数据类型 | 存储的值 | 基本操作 |
|---|---|---|---|---|---|
| STRING | 存储字符串、整数或浮点数 | SET、GET、DEL、其他字符串部分位置操作、整数或浮点数的自增自减 |
| LIST | 链表、节点上存储了字符串 | LPUSH、LPOP、LINDEX、RPUSH、RPOP、LRANG |
| SET | 集合 | SADD、SREM、SMEMBERS、SISMEMBERS|
| HASH | Key-Value无序散列表 | HSET、HGET、HGETALL、HDEL |
| ZSET | 字符串+浮点数分值，按照分值的升序分布 | ZADD、ZRANGE、ZRANGEBYSCORE、ZREM |

