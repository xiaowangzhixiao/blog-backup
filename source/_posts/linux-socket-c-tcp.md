---
title: linux下C语言socket tcp通信
date: 2018-03-17 14:58:53
tags:
- linux
- socket
category:
- 编程
- C语言
---


## 流程

### 服务器端

#### 1. 创建socket
linux中，socket被认为是一个文件，使用文件描述符操控，首先定义一个变量用来存储socket的文件描述符：sockfd。使用以下函数创建socket：
```c
	    sockfd = socket(AF_INET,SOCK_STREAM,0);
	    int socket (int __domain, int __type, int __protocol);//返回sockfd
```
int \_\_domain: 协议域，AF_INET(IPV4)、AF_INET6(IPV6)、AF_LOCAL（或称AF_UNIX，Unix域socket）、AF_ROUTE，协议域决定了socket的ip地址的类型。
int \_\_type: 制定SOCKET的类型，有SOCK_STREAM(数据流),SOCK_DGRAM(报文)
int \_\_protocol: 指定协议，通常0，使用默认协议，若type为数据流时，默认tcp，报文时默认UDP
<!-- more -->
#### 2. 绑定
```c
       int bind (int __fd, const struct sockaddr * __addr, socklen_t __len)
```
int \_\_fd socket文件描述符
const struct sockaddr * \_\_addr ip地址，其结构体有两种形式，一个是ipv4的一个是ipv6的
ipv4对应的是：
```c
     struct sockaddr_in {
        sa_family_t    sin_family; /* address family: AF_INET */
        in_port_t      sin_port;   /* port in network byte order */
        struct in_addr sin_addr;   /* internet address */
    };

    /* Internet address. */
    struct in_addr {
        uint32_t       s_addr;     /* address in network byte order */
    };
```
ipv6对应的是：
```c
         struct sockaddr_in6 {
            sa_family_t     sin6_family;   /* AF_INET6 */
            in_port_t       sin6_port;     /* port number */
            uint32_t        sin6_flowinfo; /* IPv6 flow information */
            struct in6_addr sin6_addr;     /* IPv6 address */
            uint32_t        sin6_scope_id; /* Scope ID (new in 2.4) */
        };

        struct in6_addr {
            unsigned char   s6_addr[16];   /* IPv6 address */
        };
```
socklen_t \_\_len 地址长度

#### 3. 监听
开启相应的端口，开始监听，非阻塞的。
```c
    /* Prepare to accept connections on socket FD.
       N connection requests will be queued before further requests are refused.
       Returns 0 on success, -1 for errors.  */
    int listen (int __fd, int __n)
```
fd 文件描述符
n 相应socket可以排队的最大连接个数

#### 4. accept()
收取TCP的请求，完成三次握手过程
```c
    accept (int __fd, struct sockaddr * __addr, socklen_t *__restrict __addr_len);
```
fd监听的套接字
addr 传入指针，传出客户端的地址信息
len addr的长度
返回一个连接connect_fd，使用该文件描述符进行读写，即可进行tcp通信<br>
accept默认会阻塞进程，直到有一个客户连接建立后返回，它返回的是一个新可用的套接字，这个套接字是连接套接字。
此时我们需要区分两种套接字:
1. 监听套接字: 监听套接字正如accept的参数sockfd，它是监听套接字，在调用listen函数之后，是服务器开始调用socket()函数生成的，称为监听socket描述字(监听套接字)
2. 连接套接字：一个套接字会从主动连接的套接字变身为一个监听套接字；而accept函数返回的是已连接socket描述字(一个连接套接字)，它代表着一个网络已经存在的点点连接。
一个服务器通常通常仅仅只创建一个监听socket描述字，它在该服务器的生命周期内一直存在。内核为每个由服务器进程接受的客户连接创建了一个已连接socket描述字，当服务器完成了对某个客户的服务，相应的已连接socket描述字就被关闭。

#### 5. 读写通信

本节是TCP连接，所以可以使用write和read，通过文件描述符来进行通信，除此之外还可以使用socket提供的send()，recv()函数进行通信。

|函数    |    增加的特性|
|:--|:--|
|write     |   最简单的套接口写入函数|
|send     |   增加了flags标记
|sendto   |     增加了套接口地址与套接口长度参数
|writev    |    没有标记与套接口地址，但是具有分散写入的能力
|sendmsg     |   增加标记，套接口地址与长度，分散写入以及附属数据的能力

#### 6. 关闭连接
连接套接字是需要关闭的，当结束服务时，调用close(fd)关闭连接socket。
头文件unistd.h。

## 客户端
#### 1. 创建socket
#### 2. connect() 创建连接
```c
    /* Open a connection on socket FD to peer at ADDR (which LEN bytes long).
       For connectionless socket types, just set the default address to send to
       and the only address from which to accept transmissions.
       Return 0 on success, -1 for errors.

       This function is a cancellation point and therefore not marked with
       __THROW.  */
    extern int connect (int __fd, const struct sockaddr * __addr, socklen_t __len);
```
fd: 刚刚创建的socket
addr: sockaddr_in或sockaddr_in6的结构体
len: 第二个参数的长度

#### 3. 读写操作

## 辅助函数
#### 1. IP地址相关函数

1. inet_pton
```c
        /* Convert from presentation format of an Internet number in buffer
           starting at CP to the binary network format and store result for
           interface type AF in buffer starting at BUF.  */
           int inet_pton (int __af, const char * __cp,
                      void *  __buf)
```
    af: ip类型 AF_INET, AF_INET6, AF_LOCAL...
    cp: 字符串，使用点分十进制描述的ip地址字符串
    buf: 转化完成二进制的ip地址的指针 直接传入&servaddr.sin_addr.sin_addr即可
    如果函数出错将返回一个负值，并将errno设置为EAFNOSUPPORT，如果参数af指定的地址族和cp格式不对，函数将返回0。

2. inet_ntop
```c
        /* Convert a Internet address in binary network format for interface
       type AF in buffer starting at CP to presentation form and place
       result in buffer of length LEN astarting at BUF.  */
        const char *inet_ntop (int __af, const void * __cp,
              char * __buf, socklen_t __len)
```
    和上面的函数相反

#### 2. 字节流相关函数
```c
        /* Functions to convert between host and network byte order.

       Please note that these functions normally take `unsigned long int' or
       `unsigned short int' values as arguments and also return them.  But
       this was a short-sighted decision since on different systems the types
       may have different representations but the values are always the same.  */

        extern uint32_t ntohl (uint32_t __netlong) __THROW __attribute__ ((__const__));
        extern uint16_t ntohs (uint16_t __netshort)
             __THROW __attribute__ ((__const__));
        extern uint32_t htonl (uint32_t __hostlong)
             __THROW __attribute__ ((__const__));
        extern uint16_t htons (uint16_t __hostshort)
             __THROW __attribute__ ((__const__));
```

