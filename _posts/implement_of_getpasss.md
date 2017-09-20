---
title: getpasss的实现
date: 2016-10-09 16:42:49
tags: apue
categories: Linux编程
toc: true
---
## APUE 11.10 getpass函数的实现

1. 注意点
- 调用函数`ctermid`打开控制终端，而不是直接将`/dev/tty`写在程序中。
- 只是读、写控制终端，如果不能以读、写方式打开此设备则出错返回。
- 阻塞两个信号`SIGINT`和`SIGTSTP`。如果不这样做，则在输入`INTR`字符时就会使程序终止，**并使终端仍处于禁止回送状态**。与此相类似，输入`SUSP`字符时将使程序暂停，并且在禁止回送状态下返回到shell。
- 最多只可取8个字符作为口令。输入的多余字符则被忽略。

<!--more-->

2. 源码
```
#include <signal.h>
#include <stdio.h>
#include <termios.h>
#include <stdlib.h>

#define MAX_PASS_LEN 8

char *getpass(const char *prompt)
{
    static char buf[MAX_PASS_LEN + 1];
    char *ptr;
    sigset_t sig, sigsave;
    struct termios term, termsave;
    FILE *fp;
    int c;

    if ( (fp = fopen(ctermid(NULL), "r+")) == NULL )
        return NULL;
    setbuf(fp, NULL);	//set not buffer

    /*block sigint & sigtstp*/
    sigemptyset(&sig);
    sigaddset(&sig, SIGINT);
    sigaddset(&sig, SIGTSTP);
    sigprocmask(SIG_BLOCK, &sig, &sigsave);

    tcgetattr(fileno(fp), &termsave);
    term = termsave;
    term.c_lflag &= ~(ECHO | ECHOE | ECHOK | ECHONL);
    tcsetattr(fileno(fp), TCSAFLUSH, &term);

    fputs(prompt, fp);//output prompt

    ptr = buf;
    while ( (c = getc(fp)) != EOF && c != '\n' )
    {
        if (ptr < &buf[MAX_PASS_LEN])	//abandon others character
        {
            *ptr = c;
            ptr++;
        }
    }
    *ptr = '\0';
    putc('\n', fp);

    tcsetattr(fileno(fp), TCSAFLUSH, &termsave);
    sigprocmask(SIG_SETMASK, &sigsave, NULL);

    fclose(fp);

    return buf;
}

int main()
{
    char *pass;

    pass = getpass("Please input the password: ");

    printf("PASS: %s\n", pass);

    exit(0);
}
```
