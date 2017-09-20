---
title: sleep函数的几种实现
date: 2016-10-14 18:06:33
tags: apue
categories: Linux编程
toc: true
---
使用select、polsignal三种方式实现sleep函数
<!--more-->
## 1.使用select实现
```
/*
	使用select实现sleep函数
	
	int select (int maxfdp1, fd_set *readfds, fd_set *writefds, 
			fd_set *exceptfds, struct timeval *tvptr);
	
	struct timeval{
		long tv_sec; // seconds
		long tv_usec; // and microseconds 
	};
*/
#include <sys/types.h>	/* fd_set data type */
#include <sys/time.h>	/* struct timval */
#include <stddef.h>		/**/
#include <ourhdr.h>

#define MAGNIFICATION 1000000

void sleep(unsigned int nusecs)
{
	struct timeval tval;

	tval.tv_sec = nusecs / MAGNIFICATION;
	tval.tv_sec = nusecs % MAGNIFICATION;

	/*接受到信号或者时间到期*/
	select(0, NULL, NULL, &tval);
}
```

<!--more-->

## 2.使用poll实现
```
/*
	使用poll实现sleep函数(time 为毫秒)
	
	int poll(struct pollfd fdarry[], unsigned lonng fds, int timeout);

	struct pollfd {
		int fd; // file descriptor to check, or < 0 to ignore
		short events; //events of interest on fd
		short revents;	// events that occurred on fd
	} ;
*/

#include <stropts.h>
#include <poll.h>

#define MAGNIFICATION 1000

void sleep(unsigned int nusecs)
{
	struct pollfd dummy;
	int timeout;

	if ( (timeout = nusecs / MAGNIFICATION) <= 0)
		timeout = 1;

	/*接受到信号或者时间到期*/
	poll(&dummy, 0, timeout);

}
```

## 3.使用signal实现
```
/*
	使用sigation实现sleep函数
*/


#include <signal.h>
#include <stddef.h>
#include <ourhdr.h>

static void sig_alrm(void)
{
	return; //nothing to do
}

unsigned int sleep(unsigned int nsecs)
{
	struct sigaction newact, oldact;
	sigset_t newmask, oldmask, suspmask;
	unsigned int unslept;

	newact.sa_handler = sig_alrm;
	sigemptyset(&newact.sa_mask);
	newact.sa_falgs = 0;
	sigaction(SIGALRM, &newact, &oldact);

	
	sigemptyset(&newmask);
	sigaddset(&newmask, SIGALRM);

	/*block SIGALRM and save current signal mask*/
	sigprocmask(SIG_BLOCK, &newmask, &oldmask);

	alarm(nsecs);

	suspmask = oldmask;
	sigadelset(&suspmask, SIGALRM);	//make sure SIGALRM isn't blocked

	sigsuspend(&suspmask);	//wait for ang signal to be caught

	unslept = alarm(0);
	sigaction(SIGALRM, &oldact, NULL);	//reset previous action
	sigprocmask(SIG_SETMASK, &oldmask, NULL);

	return unslept;
}
```
