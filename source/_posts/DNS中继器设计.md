---
title: DNS中继器设计
toc: true
comments: true
date: 2020-02-14 09:49:24
tags:
- 计算机网络
- DNS
category:
- 编程
- C语言
---

## 简介
现在总结一下以前大三的时候用C语言撸的一个DNS服务器的设计，源码地址为[DNS-github](https://github.com/xiaowangzhixiao/DNS)，题目要求如下：

1. 基本功能

    设计一个DNS服务器程序，读入“域名-IP地址”对照表，当客户端查询域名对应的IP地址时，用域名检索该对照表，三种检索结果： 
    1. 检索结果为ip地址0.0.0.0，则向客户端返回“域名不存在”的报错消息（不良网站拦截功能） 
    2. 检索结果为普通IP地址，则向客户返回这个地址（服务器功能） 
    3. 表中未检到该域名，则向因特网DNS服务器发出查询，并将结果返给客户端（中继功能）

1. 多客户端并发

    允许多个客户端（可能会位于不同的多个计算机）的并发查询，即：允许第一个查询尚未得到答案前就启动处理另外一个客户端查询请求（DNS协议头中ID字段的作用）

3. 超时处理

    由于UDP的不可靠性，考虑求助外部DNS服务器（中继）却不能得到应答或者收到迟到应答的情形

## 问题分析
- 题目要求完成一个基于UDP的DNS中继服务器，首先要构造一个UDP报文收发和处理框架，其次，需要对DNS的报文进行解析，这里可以使用C语言的位域很方便地将报文的每个标志位取出来，并进行相应地修改。
- 当DNS服务器没有检索到服务器时，需要向上级DNS发起请求，所以需要缓存下客户端的请求和向上级发起的请求的ID，等待上级DNS服务器的应答。
- 向上级服务器的请求可能没有回应，所以需要定时清除客户端请求的缓存。
<!-- more -->
## 了解DNS
DNS全名叫做域名解析服务器，能够将域名解析为IP地址，用户只需要记住网站的域名而不是ip就可以访问网站。
### 域名结构
一个域名是一串由.分割的字符串，从右往左的顺序划分等级，例如“www.baidu.com”，com是顶级域名，baidu是二级域名，www为三级域名。
### 服务器
DNS服务器一般分三种，根DNS服务器，顶级DNS服务器，权威DNS服务器。我们要进行开发是这三种之外的本地DNS服务器，本地服务器可以直接查询本地缓存，也可以向权威服务器查询后返回消息

### DNS解析过程

1. 当用户输入域名www.baidu.com后，首先浏览器会在浏览器DNS缓存中查询，如果有就返回ip
2. 如果没有，操作系统会先检查自己本地的hosts文件是否有这个网址的映射关系，如果有，就返回IP，完成域名解析
3. 如果还没有，我的电脑就要向本地DNS服务器发起请求查询www.baidu.com这个域名。
4. 本地DNS服务器也会查询自己的本地缓存，如果有，就直接返回，会标记为非权威回答，如果没有，就直接找根服务器，根服务器会返回com这一级的DNS服务器的ip地址，然后本地DNS拿着IP地址继续请求，如此反复，得到域名的ip地址


## 整体架构
- 主程序模块

    主程序模块主要负责控制整个程序的流程。
- 哈希表模块

    构建一个基本的哈希表结构，为域名-ip缓存和ip-消息映射提供基础结构。
- 域名-ip缓存表模块

    将预设的域名-ip文件读入本表供DNS查询。
- 网络通信模块

    提供基本的UDP报文收发功能，介绍见[Linux下UDP通信](https://www.xiaowangzhi.xyz/2018/03/19/linux-socket-c-udp/)

- DNS报文处理模块

    提供DNS报文解析，处理，构造功能。
- Ip-消息映射模块

    当客户端发来的DNS查询需要询问上层DNS服务器时，记录下客户端ip，提问报文序号和中继服务器的提问序号，以便收到DNS回答时可以正确映射到相应的提问客户端。



## 哈希表
使用C语言手撸了一套哈希表，用于对IP-消息映射和域名-IP缓存的存储。本项目中实现的哈希表的桶个数是在创建时设的固定大小，需要给定hit值指导桶的选择，使用的是拉链法解决的冲突，具体的就不在这里详细介绍了。

## 主程序
主程序框架中，首先进行初始化的配置，然后使用UDP的recvfrom的同步阻塞方法，等待客户端的请求，来一个请求，就处理一个请求，没有用多线程等技术。

## DNS解析模块

我们来看一下DNS协议的描述，这里使用位域将标志位取出来
```c
/*

The header contains the following fields:

                                    1  1  1  1  1  1
      0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                      ID                       |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |QR|   Opcode  |AA|TC|RD|RA|   Z    |   RCODE   |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    QDCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    ANCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    NSCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
    |                    ARCOUNT                    |
    +--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

ID              A 16 bit identifier assigned by the program that
                generates any kind of query.  This identifier is copied
                the corresponding reply and can be used by the requester
                to match up replies to outstanding queries.

QR              A one bit field that specifies whether this message is a
                query (0), or a response (1).

OPCODE          A four bit field that specifies kind of query in this
                message.  This value is set by the originator of a query
                and copied into the response.  The values are:

                0               a standard query (QUERY)

                1               an inverse query (IQUERY)

                2               a server status request (STATUS)

                3-15            reserved for future use

AA              Authoritative Answer - this bit is valid in responses,
                and specifies that the responding name server is an
                authority for the domain name in question section.

                Note that the contents of the answer section may have
                multiple owner names because of aliases.  The AA bit
                corresponds to the name which matches the query name, or
                the first owner name in the answer section.

TC              TrunCation - specifies that this message was truncated
                due to length greater than that permitted on the
                transmission channel.

RD              Recursion Desired - this bit may be set in a query and
                is copied into the response.  If RD is set, it directs
                the name server to pursue the query recursively.
                Recursive query support is optional.

RA              Recursion Available - this be is set or cleared in a
                response, and denotes whether recursive query support is
                available in the name server.

Z               Reserved for future use.  Must be zero in all queries
                and responses.

RCODE           Response code - this 4 bit field is set as part of
                responses.  The values have the following
                interpretation:

                0               No error condition

                1               Format error - The name server was
                                unable to interpret the query.

                2               Server failure - The name server was
                                unable to process this query due to a
                                problem with the name server.

                3               Name Error - Meaningful only for
                                responses from an authoritative name
                                server, this code signifies that the
                                domain name referenced in the query does
                                not exist.

                4               Not Implemented - The name server does
                                not support the requested kind of query.

                5               Refused - The name server refuses to
                                perform the specified operation for
                                policy reasons.  For example, a name
                                server may not wish to provide the
                                information to the particular requester,
                                or a name server may not wish to perform
                                a particular operation (e.g., zone
                                transfer) for particular data.

                6-15            Reserved for future use.

QDCOUNT         an unsigned 16 bit integer specifying the number of
                entries in the question section.

ANCOUNT         an unsigned 16 bit integer specifying the number of
                resource records in the answer section.

NSCOUNT         an unsigned 16 bit integer specifying the number of name
                server resource records in the authority records
                section.

ARCOUNT         an unsigned 16 bit integer specifying the number of
                resource records in the additional records section.



*/

typedef struct
{
	uint16_t id;                     //标志

	uint16_t RCODE : 4;          //返回码 表明返回包的类型

	uint16_t Z : 3;               //0

	uint16_t RA : 1;             //如果名字服务器支持递归，则在响应中将该比特位置1

	uint16_t RD : 1;             //recursion desired，期望递归be set in a query and is copied into the response.

	uint16_t TC : 1;             //可截断的 (truncated)

	uint16_t AA : 1;             //authoritative answer，回应中有效，表明该服务器是权威回答

	uint16_t OPCODE : 4;         //请求中有效，表明请求的类型

	uint16_t QR : 1;             //Q or R

	uint16_t QDCOUNT;
	uint16_t ANCOUNT;
	uint16_t NSCOUNT;
	uint16_t ARCOUNT;
} DNSHeader;


/*Question section format

		The question section is used to carry the "question" in most queries,
i.e., the parameters that define what is being asked.  The section
contains QDCOUNT (usually 1) entries, each of the following format:

1  1  1  1  1  1
0  1  2  3  4  5  6  7  8  9  0  1  2  3  4  5
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                                               |
/                     QNAME                     /
/                                               /
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     QTYPE                     |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|                     QCLASS                    |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

where:

		QNAME           a domain name represented as a sequence of labels, where
		each label consists of a length octet followed by that
		number of octets.  The domain name terminates with the
zero length octet for the null label of the root.  Note
		that this field may be an odd number of octets; no
padding is used.

QTYPE           a two octet code which specifies the type of the query.
The values for this field include all codes valid for a
		TYPE field, together with some more general codes which
		can match more than one type of RR.

QCLASS          a two octet code that specifies the class of the query.
For example, the QCLASS field is IN for the Internet.
*/
typedef struct
{
	char * buff;
	DNSHeader dnsHeader;
	char * host;
	uint16_t QTYPE;
	uint16_t QCLASS;
	int questionLength;
	int answerOffset;
	uint32_t ip;
	size_t size_n;
}DNS;

DNS DNS_getHead(DNS dns)
{
	uint16_t temp;
	dns.dnsHeader.id = ntohs(*(uint16_t *)dns.buff);
	memcpy(&temp,dns.buff+2,sizeof(temp));
	temp = ntohs(temp);
	memcpy(((char *)&(dns.dnsHeader))+2,&temp,sizeof(temp));
	dns.dnsHeader.QDCOUNT = ntohs(*(uint16_t *)(dns.buff + 4));
	dns.dnsHeader.ANCOUNT = ntohs(*(uint16_t *)(dns.buff + 6));
	dns.dnsHeader.NSCOUNT = ntohs(*(uint16_t *)(dns.buff + 8));
	dns.dnsHeader.ARCOUNT = ntohs(*(uint16_t *)(dns.buff + 10));
	return dns;
}
```

在解析的过程中，注意使用了ntohs函数，将网络传输的字节顺序转换为本地机器的字节顺序。这里涉及一个小知识，字节存储顺序在不同环境下是不同的，根据字节顺序分为小端存储和大端存储，小端指的是低位字节存低地址，高位字节存高地址，大端则是相反。在系统函数中直接提供了网络字节顺序和本地字节顺序的转换。

```c
htonl()//--"Host to Network Long"	长整型数据主机字节顺序转网络字节顺序
ntohl()//--"Network to Host Long"	长整型数据网络字节顺序转主机字节顺序
htons()//--"Host to Network Short"  	短整型数据主机字节顺序转网络字节顺序
ntohs()//--"Network to Host Short"  	短整型数据网络字节顺序转主机字节顺序
```

## 定时删除过期缓存

在每次循环时，使用调用IdMap_update函数对过期的缓存进行删除

```c
typedef struct{
	uint16_t **idArray;
	int length;
}IdArray;

void apply(const void *key, void **value, void *c1)
{
	time_t timeNow = time(NULL);
	uint16_t **idArray = ((IdArray *)c1)->idArray;
	IpId *ipId = *value;
	assert(ipId);
	if (timeNow - ipId->requireTime > AGE)
	{
	    idArray[((IdArray *)c1)->length++] = key;
	}
}

void IdMap_update(IdMap idMap)
{
	IdArray * idArray = malloc(sizeof(IdArray));
	idArray->length = 0;
	idArray->idArray = malloc(sizeof(uint16_t *)*HashTable_length(idMap));

	HashTable_map(idMap,apply,idArray);

	for (int i = 0; i < idArray->length; ++i)
	{
		HashTable_remove(idMap,idArray->idArray[i]);
	}

	free(idArray->idArray);
	free(idArray);
}
```

