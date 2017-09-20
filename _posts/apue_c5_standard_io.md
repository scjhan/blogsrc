---
title: chapter5 标准IO库
date: 2016-09-03 23:48:06
tags: apue
categories: Linux编程
toc: true
---
《Advanced Programming in the UNIX Environment》第五章——文标准IO库课后习题答案
<!--more-->
# Exercise 5.1
用setvbuf完成setbuf。
**Ans**  
```
//int  setvbuf(FILE *fp, char *buf, int mode, size_t size)
int SetBuf(FILE *fp, char *buf)
{
    if (NULL == fp) return -1;

    if (NULL == buf)
        if ( setvbuf(fp, buf, _IONBF, 0) != 0 )
        {
            printf("setvbuf error(buf == NULL)");
            return -1;
        }
    else
    {
        if (fp == stderr)
        {
            if ( setvbuf(fp, buf, _IONBF,BUFSIZ) !=0 )
            {
                printf("setvbuf error(fp == stderr)");
                return -1;
            }
        }
        else if (fp == stdin || fp == stdout)
        {
            if ( setvbuf(fp, buf, _IOLBF, BUFSIZ) != 0 )
            {
                printf("setvbuf error(fp == stdin || fp == stdout)");
                return -1;
            }
        }
        else
        {
            if (setvbuf(fp, buf, _IOFBF, BUFSIZ) != 0 )
            {
                printf("setvbuf error(others side)");
                return -1;
            }
        }
    }

    return 0;
}
```

****

# Exercise 5.4
下面的代码在一些机器上运行正确，而在另外一些机器运行时出错，解释问题所在。
```
#include <stdio.h>

int main(void)
{
	char c;
	while ( (c = getchar()) != EOF )
		putchar(c);
	exit(0);
}
```
**Ans**  
这是一个比较常见的错误。getc以及getchar的返回值是整型，而不是字符型。由于EOF经常定义为－1，那么如果系统使用的是有符号的字符类型，程序还可以正常工作。但如果使用的是无符号字符类型，那么返回的EOF被保存到字符c后将不再是－1，所以，程序会进入死循环。  
> int getchar(void);  
> int getc(FILE *fp);  
> 若成功则为下一个字符，若已处文件尾端或出错则为EOF  




