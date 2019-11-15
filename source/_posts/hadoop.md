---
title: hadoop学习之hdfs
date: 2018-03-22 22:00:56
tags:
- 大数据
category:
- 编程
- 大数据
---


## 优点
* 高容错性
	* 数据自动保存多个副本
	* 副本丢失后，自动恢复
* 适合批处理
	- 移动计算而非数据
	- 数据位置暴露给计算框架
- 适合大数据处理
	- GB、TB、甚至PB级别的数据
	- 百万规模以上的文件数量
	- 10K+节点
- 可构建在廉价的机器上
	- 通过多副本提高可靠性
	- 提供了容错和恢复机制

<!--more-->

## 缺点
- 低延迟数据访问
	- 毫秒级
	- 低延迟与高吞吐率
- 小文件存取
	- 占用NameNode大量的内存
	- 寻道时间超过读取时间
- 并发写入、文件随机修改
	- 一个文件只能有一个写着
	- 仅支持append

## 架构

![HDFS架构图](/img/hdfs架构.PNG)

## HDFS数据存储单元
- 文件被切分为固定大小的数据块 block 存储在不同的节点
- 默认情况下每个block都有三个副本
- block的大小和副本数通过client端上传文件时设置，文件上传成功后副本数就可变更，block size不可变更
- 追加时为新增一个block

## HDFS设计思想

![设计思路](/img/hdfs设计思想.PNG)

## NameNode
- 功能：接收客户端的读写服务
- 保存元数据 metadata
	- 文件的ouwership和permissions
	- 文件包含哪些块
	- block保存在哪个DataNode（有DataNode启动时上报）
- matedata存储到磁盘文件名为“fsimage”
- block的位置信息不会保存到磁盘
- edits文件记录metadata的操作日志
- 操作metadata是在内存中做的

## SecondaryNameNode
- 备份
- 帮助NN合并edits，减少NN启动时间
	- 合并时机  配置的时间间隔、edits文件大小达到阈值
	- ![合并](/img/edits合并.PNG)

## DataNode
- 存储数据
- 启动DN线程的时候向NN汇报block信息
- 通过向NN发送心跳保存与其联系（三秒一次），如果NN十分钟没有收到DN的心跳，则认为其已经lost，并copy其上的block到其他DN

## Block存放
1. 放置在不太满的节点上
2. 放置在第二个机架上
3. 和第二个副本相同机架上的某节点

## HDFS写流程
![写流程](/img/hdfs写流程.PNG)
![读流程](/img/HDFS读流程.PNG)

## HDFS文件权限
- 和linux文件权限类似
- 不做密码认证

## 安全模式

## 安装
## 配置
安装目录下的/etc
注意tmpdir配置
## 启动
1. namenode上格式化：`hdfs namenode -format`
2. start-dfs.sh

## HDFS 2.x解决的1.x的问题
### 内存受限问题
- HDFS Federation
- 水平扩展 支持多个NameNode
- 每个NameNode分管一部分目录
- 所有NameNode共享所有DataNode存储资料

### 单点故障问题
实现NameNode的HA（高可用）
![高可用](/img/HDFS2.0HA.PNG)

1. 两个NN，一个active，一个standby
2. edits文件和fsimage文件要在两个NN中一致---共享---edits上传到JournalNodes集群中
	1. journalNode对两个NN合并edits和fsimage
	2. DN对active和standby报告block信息，但是只响应来自activeNN的命令
3. 有FalloverController主机监控NN监控状态并通过远程命令的方式切换NameNode状态
4. FC主机向ZooKeeper汇报NN状态，ZK投票选举出active的NN
