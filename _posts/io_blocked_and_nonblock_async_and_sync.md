---
title: IO阻塞/非阻塞与同步/异步
toc: true
date: 2017-05-05 22:10:29
tags:
	- nonblocking
	- blocking
categories: 网络编程
---
### 1.阻塞(blocking)与非阻塞(nonblocking)
阻塞和非阻塞关注的是程序在等待调用结果（消息，返回值）时的状态。
<!--more-->
阻塞指系统调用返回之前，当前进程会被挂起（进入非可执行状态，CPU不会分配时间片）。函数直到有了结果才返回。
非阻塞指系统调用没有得到结果，不会阻塞当前进程，而是直接返回，同时伴随返回相应的错误提示，如EWOULDBLOCK，EAGIN。

### 2.同步(synchronous)与异步(asynchronous)
同步和异步关注的是消息通信机制(synchronous communication/ asynchronous communication)。
- 同步过程中进程触发IO操作并等待或者轮询的去查看IO操作是否完成。
例如read()调用等待读取完成返回（阻塞），或者read()直接返回不断轮询直到返回结果指示读取完成（非阻塞）。
- 异步过程中进程触发IO操作以后，直接返回，做自己的事情，IO交给内核来处理，完成后内核通知进程IO完成。

### 3. 陈硕的解释
> 在处理 IO 的时候，阻塞和非阻塞都是同步 IO。只有使用了特殊的 API 才是异步 IO。

![这里写图片描述](http://img.blog.csdn.net/20170505220755995?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvU3RyYW5nZXJfQ0pIYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
