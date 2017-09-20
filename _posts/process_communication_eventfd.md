---
title: 使用eventfd进行进程/线程通信
toc: true
date: 2017-05-03 16:15:06
tags: eventfd
categories: 网络编程
---
eventfd在Linux中类似于管道，但是比管道简单。他的收发缓存只有8字节。使用eventfd通信时，通过evenfd(2)创建一个文件描述符，主要用于事件通知(wait/notify)，传输数据只能是8个字节大小，一般用`uint64_t`，是一个双方协商好的数字。
<!--more-->

### eventfd(2)
eventfd(2)用于创建一个描述符用于事件通知，其原型如下：
```cpp
#include <sys/eventfd.h>

int eventfd(unsigned int initval, int flags);
```
- initval参数用于初始化eventfd内部计数器（见下）
- flags参数用于设定文件描述符标志
 - EFD_NONBLOCK
 - EFD_CLOEXEC
 - EFD_SEMAPHORE

### eventfd原理
eventfd(2)返回一个文件描述符，并在内核中关联一个对应的struct file结构。这个struct file结构如下：
```cpp
struct eventfd_ctx {
	struct kref kref;	//linux kernel实现file计数用的
	wait_queue_head_t wqh;	//Linux kernel等待队列
	/*
	* Every time that a write(2) is performed on an eventfd, the
	* value of the __u64 being written is added to "count" and a
	* wakeup is performed on "wqh". A read(2) will return the "count"
	* value to userspace, and will reset "count" to zero. The kernel
	* side eventfd_signal() also, adds to the "count" counter and
	* issue a wakeup.
	*/
	__u64 count;		//eventfd计数器
	unsigned int flags;	//文件描述符标志
};
```
- read一个eventfd时，从count读出数据，然后将count清零
- write一个eventfd时，将数据写入count

### eventfd应用场景
eventfd传输数据只有八个字节，注定了不是用来进行数据传输的。由于eventfd不用自己管理缓存区，因此可以非常搞笑的实现唤醒，即：event wait/notify。
例如主线程阻塞在epoll_wait等待返回就绪描述符列表，此时在另外一个线程中修改某个对象，但不是描述符，然而希望epoll_wait能够返回并让程序继续往下执行，这个时候就需要一种机制来唤醒该线程。
具体实现如下：
其他线程修改对象之后向eventfd写入数据，触发eventfd的可读事件，让epoll_wait返回，这个时候阻塞线程就成功返回了。

### eventfd进程通信示例
```cpp
#include <sys/eventfd.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <stdint.h>	/* Definition of uint64_t */

#define handle_error(msg) \
	do { perror(msg); exit(EXIT_FAILURE); } while (0)

int main(int argc, char *argv[])
{
	int efd, j;
	uint64_t u;
	ssize_t s;

	if (argc < 2) {
		fprintf(stderr, "Usage: %s <num>...\n", argv[0]);
		exit(EXIT_FAILURE);
	}

	efd = eventfd(0, 0);
	if (efd == -1)
	handle_error("eventfd");

	switch (fork()) {
	case 0:
	for (j = 1; j < argc; j++) {
		printf("Child writing %s to efd\n", argv[j]);
		u = strtoull(argv[j], NULL, 0);
		strtoull() allows various bases */
		s = write(efd, &u, sizeof(uint64_t));
		if (s != sizeof(uint64_t))
		handle_error("write");
	}
	printf("Child completed write loop\n");

	exit(EXIT_SUCCESS);

	default:
		sleep(2);

		printf("Parent about to read\n");
		s = read(efd, &u, sizeof(uint64_t));
		if (s != sizeof(uint64_t))
			handle_error("read");
		printf("Parent read %llu (0x%llx) from efd\n",
		(unsigned long long) u, (unsigned long long) u);
		exit(EXIT_SUCCESS);

	case -1:
		handle_error("fork");
	}
}
```
