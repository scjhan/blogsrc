---
title: apue chapter3 Files IO
date: 2016-08-29 15:50:47
tags: apue
categories: 网络编程
toc: true
---
《Advanced Programming in the UNIX Environment》第三章——文件IO课后习题答案

<!--more-->

## Exercise3.2
编写一个同3.12节中的dup2功能相同的函数，要求不调用fcntl函数并且要有正确的出错处理。
**思路：递归调用dup**
```c
#include <ourhdr.h>
#include <fcntl.h>

int dup_2(int, int);

int main(void)
{
    //dup_2(1,10);
}

/* if success
 *      return the filedes
 * else
 *      return -1
 */
int dup_2(int filedes, int filedes2)
{
    if (filedes2 < 0) return -1;

    if (filedes == filedes2) return filedes2;

    if (close(filedes2) == -1) return -1;

    int newfd;
    if ((newfd = dup(filedes)) == filedes2) return filedes;
    else
    {
        dup_2(filedes, filedes2);
        close(newfd);
    }

    return filedes2;
}
```
****
## Exercise3.3
假设一个进程执行下面的3个函数调用：
```c
fd1 = open(pathname, oflags);
fd2 = dup(fd1);
fd3 = open(pathname, oflags);
```
画出结果图（见图3-3）。对fcntl作用于fd1来说，F_SETFD命令会影响哪一个文件描述符？FSETFL呢？
**Ans**  
每次调用open函数就分配一个文件表项，如果两次打开的是相同的文件，则两个文件表项指向相同的v节点。调用dup引用已存在的文件表项（此处指fd1的文件表项），见图C-1。当F_SETFD作用于fd1时，只影响fd1的文件描述符标志；F_SETFL作用于fd1时，则影响fd1及fd2的文件描述符标志。
![图C-1](http://i2.buimg.com/567571/a2ef9282a1dda9c5.jpg)
> **F_SETFL** 将文件状态标志设置为第三个参数的值。(取为整型值)
> **F_SETFD** 对于filedes 设置文件描述符标志。

****
## Exercise3.4
在许多程序中都包含下面一段代码：
```c
dup2(fd, 0);  
dup2(fd, 1);  
dup2(fd, 2);  
if (fd > 2)  
    close(fd);  
```
为了说明if语句的必要性，假设fd是1，画出每次调用dup2时3个描述符项及相应的文件表项的变化情况。然后再画出fd为3的情况。
**Ans**  
如果fd是1，执行dup2(fd，1)后返回1，但是没有关闭描述符1（见3 .12节）。调用3次dup2后，3个描述符指向相同的文件表项，所以不需要关闭描述符。如果fd是3，调用3次dup2后，有4个描述符指向相同的文件表项，所以需要关闭描述符3。  
事实上，0、1、2分别表示标准输入，标准输出，标准错误输出，一般是不关闭的。而当`fd > 2`时，表示其余的文件描述符，一般而言需要关闭。

****
## Exercise3.5
在Bourne shell和KornShell中，digit1>&digit2表示要将描述符digit1重定向至描述符digit2的同一文件。请说明下面两条命令的区别。
```
a.out > outfile 2>&1
a.out 2>&1 > outfile
```
（提示：shell从左到右处理命令行。）
**Ans**  
shell从左到右处理命令行,所以
`a.out > outfile 2>&1`
首先设置标准输出到outfile，然后执行dups将标准输出复制到描述符2（标准错误）上，其结果是将标准输出和标准错误设置为相同的文件，即描述符1和2指向相同的文件表项。而对于命令行
`a.out 2 >&1 >outfile`
由于首先执行dups，所以描述符2成为终端（假设命令是交互执行的），标准输出重定向到outfile。结果是描述符1指向outfile的文件表项，描述符2指向终端的文件表项。
