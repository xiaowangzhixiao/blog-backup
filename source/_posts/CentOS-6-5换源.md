---
title: CentOS 6.5换源
date: 2018-03-17 13:43:49
tags: 
- linux
category: 
- 编程
- 折腾
---

## 简介
身处教育网ipv6的环境下，怎么能消耗自己的校内流量来升级安装软件呢，本文介绍给大家将CentOS的yum安装源换为可走ipv6的中科大源，大家有兴趣的话，可用尝试清华源和交大源，据说都不错。
## 步骤
访问[Centos镜像使用帮助](https://lug.ustc.edu.cn/wiki/mirrors/help/centos)查看其中介绍

1.首先备份CentOS-Base.repo `mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup`
2.下载相应版本的CentOS-Base.repo, 放入/etc/yum.repos.d/
3.运行`yum makecache`生成缓存
	
