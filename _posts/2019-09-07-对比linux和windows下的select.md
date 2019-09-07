---
layout:     post
title:      对比linux和windows下的select
subtitle:   网络编程
date:       2019-09-07
author:     YunLambert
header-img: img/post-bg-select.jpg
catalog: true
tags:
    - Tips
    - Share
    - Network
---

> 对比linux和windows下的select

在使用select的时候，一直以为linux和windows上只有`FD_SETSIZE`的限制不同而已，但是最近发现底层基本数据结构不太相同。所以在这里总结一下。

## fd_set结构

先看一下windows下的`fd_set`结构

```C++
#ifndef FD_SETSIZE
#define FD_SETSIZE      64
#endif /* FD_SETSIZE */

typedef struct fd_set {
        u_int fd_count;               /* how many are SET? */
        SOCKET  fd_array[FD_SETSIZE];   /* an array of SOCKETs */
} fd_set;
```

在这里，首先定义一个了`u_int`类型的`fd_count`用来记录连接的套接字的个数，然后定义了一个存储连接的套接字数组`fd_array`。也就是说套接字是直接存储在`fd_array`里的，局限就是64.

而在linux下的`fd_set`结构是这样的：

```C++
   #define __FD_SETSIZE    1024
   typedef __kernel_fd_set     fd_set;
   typedef struct {
       unsigned long fds_bits[__FD_SETSIZE / (8 * sizeof(long))];
   } __kernel_fd_set;
```

fd_set结构体里面是一个无符号长整型的数组，总共有1024/(8 * 4) =  32个元素，然而这并不是说select最多只能监控32个文件的变化。过去，描述符集被作为一个整数位屏蔽码得到实现，但是这种实现对于多于32个的文件描述符将无法工作。描述符集现在通常用整数数组中的**位域**表示，数组元素的每一位对应一个文件描述符。例如，一个整数占32位，那么整数数组的第一个元素代表文件描述符0到31，数组的第二个元素代表文件描述符32到63，以此类推，最多可监控32*32  = 1024个文件的变化。

通俗来讲，就是在windows上数组的一个元素只能存储一个套接字，而在linux上数组的一个元素的每一个bit都可以对应成一个文件描述符，1字节的`fd_set`最大可以对应8个fd。

## select函数原型

select函数的功能是：轮询检查套接字信号的到来，如果有可读信号，则将相应的可读套接字保存在readfds里；如果有可写信号，则将对应的可写套接字保存在writefds里。返回值：返回值 `=`0 超时， 返回值`<`0 错误，返回值`>`0正常 

### windows下的select

```C++
int
WSAAPI
select(
    _In_ int nfds,
    _Inout_opt_ fd_set FAR * readfds,
    _Inout_opt_ fd_set FAR * writefds,
    _Inout_opt_ fd_set FAR * exceptfds,
    _In_opt_ const struct timeval FAR * timeout
    );
```

实现时：

```
fd_set fdRead;  
while (true)
{
    FD_ZERO(&fdRead);//清空
    fdRead = g_fdClientSock;
    timeval tv={0,100};
    nRet = select(0, &fdRead, 0, 0, &tv);//select
    if (nRet == 0)//超时
    continue;
    else if(nRet != SOCKET_ERROR)
    {
        for (int i = 0; i < fdRead.fd_count; i++)//for取出有信号的套接字
        {
        ...
        }
	}
}
```

如果现在有2个客户端连接，套接字值分别为120,124；那么
```
g_fdClientSock.fd_count = 2;
g_fdClientSock.fd_array[0]=120;
g_fdClientSock.fd_array[1]=124;
```

然后赋值到`fdRead`中，然后对于数据的接收我们只需要轮询一遍fdRead，将有信号的套接字按照数组内的排序顺序一次放在fdRead前面。

### linux下的select

```C++
extern int select (int __nfds, fd_set *__restrict __readfds,
		   fd_set *__restrict __writefds,
		   fd_set *__restrict __exceptfds,
		   struct timeval *__restrict __timeout);
```

总体上的原理和windows上是一样的，但是在存储套接字上是有区别的：假如接入了两个客户端，套接字分别为1,2，那么存储的时候执行`FD_SET(fd, &set)`后的set就变为了`000000011`，第一个位置与第二位置是有套接字的。如果`fd = 1`上发生了可读时间，则select返回，set变为`00000001`，没有事件发生的`fd=2`将被清空。但是一般为了存储客户端的连接的套接字，会有一个存储正在连接的套接字动态数组，可以辅助`FD_SET`。

## 各自的局限

### windows的局限

1.`FD_SETSIZE`的值为64，虽然有优化方法，但是select函数一次只能轮询64个

### linux的局限

1.每次调用 select 都需要把fd集合从用户态拷贝到内核态，fd很多时开销很大
2.调用 select 需要内核遍历 fd， fd 很多时开销很大
3.select 支持文件描述符监视有数量限制，默认 1024  

