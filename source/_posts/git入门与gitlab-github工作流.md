---
title: git入门与gitlab/github工作流
toc: true
comments: true
date: 2020-01-02 23:24:01
tags:
- git
category:
- 编程
- git
---

## git入门
- git：类似svn，都是版本管理系统，不同的是svn是集中式版本管理系统，git是分布式版本管理系统，每个开发者在自己的本地都有完整的代码版本库，即使断网都可以完成代码版本的管理。
- git仓库：一个git管理的工程文件夹，在本地的文件夹都可以初始化为git仓库由git管理
- git远程仓库：位于服务器的git仓库，方便开发者之间代码的共享
- git三种状态：已修改、以暂存、已提交
![](https://pic3.zhimg.com/80/v2-79cba8d658ab321888a37eea724a34da_hd.jpg)
### 基本操作
- 初始化git仓库：`git init`
- 从远程仓库拷贝git仓库：`git clone url`
- 数据流程
![](https://pic2.zhimg.com/80/v2-5417d98f4083ded2f48cc63e6a2f8c69_hd.jpg)
- 提交日志规范
![](http://freedisk.free4inno.com/download?uuid=3546c52b-3d1d-4fb2-8c8d-28369ae7de6a)
### 分支操作
- 主分支 master
- 开发分支 dev
- 修复分支 fix
- 功能分支 xxx，xxx
### 标签 tag
一般用于版本号
### IDEA操作演示


## github/gitlab
github和gitlab是两大基于git的代码托管服务，其中gitlab是开源的，我们可以自己部署属于自己的gitlab,域名是 gitlab.free4inno.com

### 工作流
- 功能分支

![](https://pic2.zhimg.com/80/v2-006d71f1eec0a6cdc4c74dbc09157625_hd.jpg)
- GitFlow 工作流

![](https://pic1.zhimg.com/80/v2-385addf6918290661474e687e1b661b4_hd.jpg)
- fork工作流

![](https://img-blog.csdn.net/20170215235113109?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd3dqXzc0OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
### 操作演示

