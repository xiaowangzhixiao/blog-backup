---
title: linux下C语言socket udp通信
date: 2018-03-19 22:54:27
tags:
- linux
- socket
category:
- 编程
- C语言
---


## 简介
UDP协议是面向无连接的协议，一般用于一些实时性比较强的或正确率要求不高的通信中。
## 流程
上一节当中讲了linux下socket TCP编程，有了TCP的基础，UDP就好理解多了。

<!--more-->

### 接收端
1. 创建socket
2. 绑定ip和端口
3. 等待接收 使用recvfrom函数
	```c
	        /* Read N bytes into BUF through socket FD.
	           If ADDR is not NULL, fill in *ADDR_LEN bytes of it with tha address of
	           the sender, and store the actual size of the address in *ADDR_LEN.
	           Returns the number of bytes read or -1 for errors.

	           This function is a cancellation point and therefore not marked with
	           __THROW.  */
	        extern ssize_t recvfrom (int __fd, void * __buf, size_t __n,
	                     int __flags, __SOCKADDR_ARG __addr,
	                     socklen_t * __addr_len);
	```

4. close(fd) 关闭SOCKET

### 发送端

1. 创建socket
2. 发送使用sendto()函数

	```c
	        /* Send N bytes of BUF on socket FD to peer at address ADDR (which is
	           ADDR_LEN bytes long).  Returns the number sent, or -1 for errors.

	           This function is a cancellation point and therefore not marked with
	           __THROW.  */
	        extern ssize_t sendto (int __fd, const void *__buf, size_t __n,
	                       int __flags, __CONST_SOCKADDR_ARG __addr,
	                       socklen_t __addr_len);
	```

3. close(fd) 关闭SOCKET
