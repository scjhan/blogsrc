---
title: EAGAIN、EINTR、EWOULDBLOCK与非阻塞
toc: true
date: 2017-05-03 16:40:33
tags: 
	- nonblocking
	- errno
categories: 网络编程
---
在Linux环境下开发经常会碰到很多错误(设置errno)，其中EAGAIN、EINTR、EWOULDBLOCK算是比较常见的错误，特别是在非阻塞中。
<!--more-->

#### EAGIN和EWOULDBLOCK
例如对一个设置了`O_NONBLOCKING的`文件描述符read，此时没有数据可以读，read调用不会阻塞而是返回0并且设置errno为`EAGIN`，提示稍后再试。
又或者当一个系统调用(如fork())没有足够的资源而执行失败也会设置EAGIN错误，提示稍后再试。

`EWOULDBLOCK`和`EAGIN`一样，不同平台上的。

如果对`EWOULDBLOCK`和`EAGIN`的情况进行判断，那么系统默认会输出错误描述`Resource temporarily unavailable`错误并终止进程。

#### EINTR
当一个系统调用执行被中断时，errno被设置为`EINTR`，错误描述为`Interrupted system call`。
