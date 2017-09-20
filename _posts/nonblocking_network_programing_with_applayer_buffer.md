---
title: nonblocking网络编程使用应用层Buffer
toc: true
date: 2017-05-05 21:43:12
tags: 
	- nonblocking
	- socket
categories: 网络编程
---
non-blocking IO的核心思想是避免阻塞在read()或者write()或者其他IO系统调用上，这样可以最大限度地复用thread-of-control，让线程只能阻塞在IO multiplexing函数上（select、poll、epoll_wait），这样一来，每个TCP socket都要有stateful的input buffer和output buffer。

<!--more-->

### Ⅰ. 必须有output buffer
1. 假设程序想通过TCP发送100KB数据，但是write()调用受限于TCP窗口大小，系统只能接受80KB，这个时候肯定不能原地等待（要等对方确认接受数据才能滑动TCP窗口，这个时间是不确定的），此时程序应该尽快交出控制权，返回eventloop，但是剩下的20KB不能不管。
2. 所以，需要一个应用层的output buffer来保存数据，程序只管将数据写入output buffer，剩下的发送交给output buffer。也就是注册一个POLLOUT事件，一旦socket可读就将output buffer中的剩据发送出去，如果没有将output buffer中的数据全部发送出去，就应该继续关注POLLOUT事件，当output buffer中的数据全部写完之后再取消关注POLLOUT事件。

### Ⅱ. 必须有input buffer
TCP是无边界的字节流协议，接收端必须自己处理“收到一条不完整消息”和“一次收到多条消息的数据“等情况。
假设发送方程序发送了两条1KB的消息。接收端可能收到的情况有：
- 一次性2KB
- 两次收到1KB
- 一次400B，一次1400B
- 等等情况

对于接收端，每次read()调用都必须把socket一次性读取出来（搬到应用层input buffer），否则将反复触发POLLIN事件，造成busy-loop。
至于“收到一条不完整消息”和“一次收到多条消息的数据“等情况就由input buffer来处理。这种逻辑一般是codec的职责。
