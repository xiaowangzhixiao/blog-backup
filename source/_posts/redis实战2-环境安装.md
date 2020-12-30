---
title: redis实战2--环境安装
toc: true
comments: true
date: 2020-01-26 20:47:57
tags:
- 读书笔记
- redis实战
category:
- 编程
- redis
---

在Windows系统的Ubuntu 18子系统中采用源码方式安装redis和python客户端

## 安装redis
打开Linux子系统，安装编译工具
```bash
sudo apt install make gcc python-dev
```
下载源码编译安装
```
wget -q http://download.redis.io/releases/redis-5.0.7.tar.gz
tar -zxvf http://download.redis.io/releases/redis-5.0.7.tar.gz
cd redis-5.0.7/
make 
sudo make install
```
启动redis
```
redis-server redis.conf
```

## redis命令行验证
运行一下命令，启动redis命令行客户端，连接本地redis服务
```
$redis-cli
127.0.0.1:6379>
```

## 安装Python客户端
```
$sudo python3.6 -m easy_install redis hiredis
```
验证一下
```
$python3.6
>>> import redis
>>> conn = redis.Redis()
>>> conn.set('hello', 'world!')
True
>>> conn.get('hello')
b'world!'
```
