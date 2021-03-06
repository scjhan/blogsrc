---
title: 【转】Golang CSP并发模型
toc: true
comments: true
date: 2017-07-14 18:00:52
tags: csp
categories: 
	- Go
	- 并发编程
	- 转载
---

## CSP并发模型

CSP模型是上个世纪七十年代提出的，用于描述两个独立的并发实体通过共享的通讯 channel(管道)进行通信的并发模型。 CSP中channel是第一类对象，它不关注发送消息的实体，而关注与发送消息时使用的channel。

<!--more-->

## Golang CSP

Golang 就是借用CSP模型的一些概念为之实现并发进行理论支持，其实从实际上出发，go语言并没有，完全实现了CSP模型的所有理论，仅仅是借用了 process和channel这两个概念。process是在go语言上的表现就是 goroutine 是实际并发执行的实体，每个实体之间是通过channel通讯来实现数据共享。

## Channel

Golang中使用 CSP中 channel 这个概念。channel 是被单独创建并且可以在进程之间传递，它的通信模式类似于 boss-worker 模式的，一个实体通过将消息发送到channel 中，然后又监听这个 channel 的实体处理，两个实体之间是匿名的，这个就实现实体中间的解耦，其中 channel 是同步的一个消息被发送到 channel 中，最终是一定要被另外的实体消费掉的，在实现原理上其实是一个阻塞的消息队列。

## Goroutine

goroutine 是实际并发执行的实体，它底层是使用协程(coroutine)实现并发，coroutine是一种运行在用户态的用户线程，类似于 greenthread，go底层选择使用coroutine的出发点是因为，它具有以下特点：

- 用户空间 避免了内核态和用户态的切换导致的成本
- 可以由语言和框架层进行调度
- 更小的栈空间允许创建大量的实例

可以看到第二条 用户空间线程的调度不是由操作系统来完成的，像在java 1.3中使用的greenthread的是由JVM统一调度的(后java已经改为内核线程)，还有在ruby中的fiber(半协程) 是需要在重新中自己进行调度的，而goroutine是在golang层面提供了调度器，并且对网络IO库进行了封装，屏蔽了复杂的细节，对外提供统一的语法关键字支持，简化了并发程序编写的成本。

## Goroutine 调度器

上节已经说了，golang使用goroutine做为最小的执行单位，但是这个执行单位还是在用户空间，实际上最后被处理器执行的还是内核中的线程，用户线程和内核线程的调度方法有：

- N:1 多个用户线程对应一个内核线程

  ![](/uploads/gocsp_n1.png)

- 1:1 一个用户线程对应一个内核线程

  ![](/uploads/gocsp_11.png)

- M:N 用户线程和内核线程是多对多的对应关系

  ![](/uploads/gocsp_mn.png)

golang 通过为goroutine提供语言层面的调度器，来实现了高效率的M:N线程对应关系。

![](/uploads/gocsp.png)

图中

- M：是内核线程
- P :  是调度协调，用于协调M和G的执行，内核线程只有拿到了 P才能对goroutine继续调度执行，一般都是通过限定P的个数来控制golang的并发度
- G : 是待执行的goroutine，包含这个goroutine的栈空间
- Gn : 灰色背景的Gn 是已经挂起的goroutine，它们被添加到了执行队列中，然后需要等待网络IO的goroutine，当P通过 epoll查询到特定的fd的时候，会重新调度起对应的，正在挂起的goroutine。 

Golang为了调度的公平性，在调度器加入了steal working 算法 ，在一个P自己的执行队列，处理完之后，它会先到全局的执行队列中偷G进行处理，如果没有的话，再会到其他P的执行队列中抢G来进行处理。

## 总结

Golang实现了 CSP 并发模型做为并发基础，底层使用goroutine做为并发实体，goroutine非常轻量级可以创建几十万个实体。实体间通过 channel 继续匿名消息传递使之解耦，在语言层面实现了自动调度，这样屏蔽了很多内部细节，对外提供简单的语法关键字，大大简化了并发编程的思维转换和管理线程的复杂性。