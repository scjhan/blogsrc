---
title: 使用timerfd
toc: true
date: 2017-04-30 17:09:51
tags: timerfd
categories: 网络编程
---
timerfd是Linux为用户程序提供的一个定时器接口。这个接口基于文件描述符，通过文件描述符的可读事件进行超时通知，所以能够被用于select/poll的应用场景。
<!--more-->

### timerfd接口
1. timerfd_create(2)
timerfd_create(2)函数用于创建timerfd文件描述符，其函数原型如下：
```cpp
int timerfd_create(int clockid, int flags);
```
- clockid参数，设置时间获取方式
**CLOCK_REALTIME**: 系统范围内的实时时钟（以1970.1.1为基准）
**CLOCK_MONOTONIC**: 以固定的速率运行，从不进行调整和复位 ,它不受任何系统time-of-day时钟修改的影响（以开机时间为基准）
- flags参数，设置描述符属性
**TFD_NONBLOCK**: 非阻塞
**TFD_CLOEXEC**: CLOEXEC


2. timerfd_settime(4)
timerfd_settime(4)用于启动和停止计数器，原型如下：
```cpp
struct timespec
{
	time_t tv_sec;  /* Seconds */
	long   tv_nsec; /* Nanoseconds */
};
struct itimerspec
{
	struct timespec it_interval;    /* 计时周期 */
	struct timespec it_value;       /* 开始时间 */
};

int timerfd_settime(int fd, int flags, const struct itimerspec *new_value, struct itimerspec *old_value);
```
- fd参数
即timerfd_create(2)创建的timerfd描述符
- flags参数
**0** : 相对计时器，此时itimerspec参数填的是相对时间
**TFD_TIMER_ABSTIME** : 绝对定时器，此时itimerspec填的是使用clock_gettime(2)获取的时间，clock_gettime第一个参数与timerfd_create的clockid参数相同。
- new_value参数
用于指定定时器的动作，为0表示停止计时器
- old_value


3. timerfd_gettime(2)
```cpp
int timerfd_gettime(int fd, struct itimerspec *curr_value);
```
此函数用于获得定时器距离下次超时还剩下的时间。如果调用时定时器已经到期，并且该定时器处于循环模式（设置超时时间时struct itimerspec::it_interval不为0），那么调用此函数之后定时器重新开始计时。

### read读取timerfd超时事件
当定时器超时，read读事件发生即可读，返回超时次数（从上次调用timerfd_settime()启动开始或上次read成功读取开始），它是一个8字节的unit64_t类型整数，如果定时器没有发生超时事件，则read将阻塞若timerfd为阻塞模式，否则返回EAGAIN 错误（O_NONBLOCK模式），如果read时提供的缓冲区小于8字节将以EINVAL错误返回。

### timerfd实例
```cpp
#include <sys/timerfd.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

void startRealtimer(unsigned int beginSec, unsigned int periodSec, int count)
{
    int tfd = timerfd_create(CLOCK_REALTIME, TFD_CLOEXEC);
    timespec now;
    clock_gettime(CLOCK_REALTIME, &now);
    struct itimerspec new_value;
    new_value.it_value.tv_sec = now.tv_sec + beginSec;
    new_value.it_value.tv_nsec = now.tv_nsec;
    new_value.it_interval.tv_sec = periodSec;
    new_value.it_interval.tv_nsec = 0;

    timerfd_settime(tfd, TFD_TIMER_ABSTIME, &new_value, NULL);
    for (int i = 0;i < count; /* no ++i */)
    {
        unsigned long long data = 0;
        read(tfd, &data, sizeof(data));
        i += data;
        printf("Realtimer Count %d\n", i);
    }
    close(tfd);
}

void startMonotonictimer(unsigned int beginSec, unsigned int periodSec, int count)
{
    int tfd = timerfd_create(CLOCK_MONOTONIC, TFD_CLOEXEC);
    struct itimerspec new_value;
    new_value.it_value.tv_sec = beginSec;
    new_value.it_interval.tv_sec = periodSec;
    timerfd_settime(tfd, 0, &new_value, NULL);
    for (int i = 0;i < count; /* no ++i */)
    {
        unsigned long data;
        read(tfd, &data, sizeof(data));
        i += data;
        printf("Monotonictimer Count %d\n", i);
    }
    close(tfd);
}

int main()
{
    startRealtimer(5, 1, 5);

    startMonotonictimer(5, 1, 5);

    return 0;
}
```
