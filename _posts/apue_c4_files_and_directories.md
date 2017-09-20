---
title: apue chapter4 文件和目录
date: 2016-09-03 21:32:26
tags: apue
categories: Linux编程
toc: true
---
《Advanced Programming in the UNIX Environment》第四章——文件和目录课后习题答案
<!--more-->
# Exercise4.2
表4-1指出SVR4没有提供宏S_ISLNK，但是SVR4支持符号连接并且在<sys/stat.h>中定义了SIFLNK，如何修改ourhdr.h使得需要SISLNK宏的程序可以使用它？
**Ans**  
将下面的几行语句加入<ourhdr.h> 
```
#if defined (S_IFLNK)  &&  !defined(S_ISLNK )  
	#define  S_ISLNK(mode)    (((mode) & S_IFMT ) == S_IFLNK)
#endif 
```
这是一个我们编写的头文件如何屏蔽某些系统差别的实例。

****

# Exercise 4.7
编写一个类似cp(1)的程序，它复制包含空洞的文件，但不将字节0写到输出文件中去。  
**Ans**  
```
#include <sys/stat.h>
#include <sys/types.h>
#include <ourhdr.h>
#include <fcntl.h>

typedef int bool;

#define true 1
#define false 0
#define BUFFER_SIZE 8192
#define BLOCK_SIZE 512

int copy(const char *readFile, const char *writeFile)
{
    int fd1,fd2;
    char buf[BUFFER_SIZE] = {0};
    struct stat st;
    bool hasHoles = false;
    int byte_count = 0,redNum = 0;
    if ( (fd1 = open(readFile, O_RDWR)) == -1 )
    {
        err_sys("Open file error");
        return -1;
    }
    if (fstat(fd1, &st) == -1)
    {
        err_sys("fstat error");
        return -1;
    }
    else if (S_ISREG(st.st_mode) && st.st_size > BLOCK_SIZE * st.st_blocks)
        hasHoles = true;

    if ( (fd2 = open(writeFile, O_RDWR | O_APPEND | O_CREAT | O_TRUNC, 0664)) == -1)
    {
        err_sys("open write file error");
        return -1;
    }

    if ( (redNum = read(fd1, buf, BUFFER_SIZE)) == -1 )
    {
        err_sys("read error");
        return -1;
    }

    if (hasHoles)
    {
        int i = 0;
        for (i = 0;i < redNum; ++i)
        {
            buf[byte_count] = buf[i];
            if (buf[i] != '\0') ++byte_count;
        }
    }
    else byte_count = redNum;

    if (write(fd2, buf, BUFFER_SIZE) == -1)
    {
        err_sys("write file error");
        return -1;
    }

    close(fd1);
    close(fd2);

    return 0;
}

int main(int argc, char *argv[])
{
    if (argc != 3)
        err_quit("error args");
    if (copy(argv[1], argv[2]) == -1)
        err_sys("copy error");

    exit(0);

}
```
