---
title: GDB调试工具入门
toc: true
date: 2017-04-17 12:47:51
tags: gdb
categories: Linux编程
---
## 0 gdb介绍
调试器GDB允许查看在执行一个程序时其内部时发生了什么，或者是程序奔溃(crashed)时它正在做什么。
gdb通过以下四种事情来捕获某个行为的异常错误(bug)：
- 运行程序，指定可能影响其动作的内容。
- 让程序在指定的情况下停止。
- 检查当程序停止时发生了什么。
- 改变程序中的内容，以便于更正一个错误，然后继续寻找下一个错误。
<!--more-->
gdb可用于调试C，C++，Fortran，Modula-2。
gdb通过在终端执行`gdb`命令激活。一旦启动，他从命令行读取命令，直到使用`quit`命令让它退出。也可以通过使用`help command`命令获取在线帮助。
gdb可以使用无参或者带参无选项执行，不过一般都使用带参的命令，例如：指定一个程序或者指定core文件：
```python
gdb program	#指定一个可执行程序
gdb program core	#指定一个可执行程序以及core文件
gdb program pid	#指定一个正在运行的程序
gdb -p pid	#同上
```

## 1 gdb常用命令
#### 1.1 list
list命令用于查看代码，可简写为l。
```python
list	#查看上一次list中心附近的10行代码，-5～+5
list n	#查看第n行附近10行代码，n-5～n+5
list b,e	#查看b,e行范围的代码
list function	#查看函数function附近的代码
list file:line	#查看文件file第line行附件的代码
list file:function	#查看文件file的函数function附近的代码
list *address	#地址为address的行附近的代码，使用info add name获取地址
```
#### 1.2 break
break命令用于设置断点，可用b简化；delete用于删除断点，可用d简化。
```python
break n	#在第n行设置断点
break function	#在函数function设置断点，可以是库函数
break file:line	#在文件file第line行设置断点
break file:function	#在文件file的function函数设置断点
break n if condition	#根据条件在第n行设置断点，例如b 16 if i==10
break *address	#在地址为address的行设置断点
```
#### 1.3 delete和clear
每次使用break设置断点都会分配一个断点号，例如：
```shell
(gdb) b 16
Breakpoint 1 at 0x400512: file test.cc, line 16.
(gdb) b 17
Breakpoint 2 at 0x40051b: file test.cc, line 17.
```
要删除断点使用可以使用`delete`命令：
```perl
delete [breakpoints num] [range...]
delete n	#删除n号断点
delete m-n	#删除m-n号断点
```
也可以使用`clear`命令，clear是**基于行**的，不是删除所有断点：
```perl
clear n	#删除n行的所有断点
clear function	#删除函数function的断点
clear file:line	#删除文件file第n行的所有断点
clear file:function	#删除文件:函数的所有断点
```

#### 1.4 查看变量
1. `print`命令
print命令用来在调试程序时查看变量值，可简化为p。
```python
print var	#打印var的值
print *array@len	#以{a, b, ...}格式打印动态数组
print array	#以{a, b, ...}格式打印静态数组
print file::var or print function::var	#打印全局变量
#指定输出格式：
print/u[d|o|x...] ...	#作为无符号数输出
print/t ...	#二进制输出
print/d ...	#十进制输出
print/o ...	#八进制输出
print/x ...	#十六进制输出
print/a ...	#十六进制输出
print/c ...	#字符输出
print/f ...	#浮点数输出
```
2. `display`命令
```python
display var	#每次执行到断点都打印var的值
```

#### 1.5 程序执行时命令
1. `run`命令
run命令用来开始执行程序，直到第一个断点停止程序。可简写为r。例如：
```python
(gdb) break 20
Breakpoint 1 at 0x40056d: file test.cc, line 20.
(gdb) run
Starting program: /home/chenjunhan/Learning/gdb/test Breakpoint 1, main () at test.cc:20
```
2. `continue`命令
continue命令用来从上次断点停止之后开始继续执行程序，直到下一个断点。可简写为c。例如：
```python
(gdb) b 15
Breakpoint 1 at 0x40050b: file test.cc, line 15.
(gdb) b 16
Breakpoint 2 at 0x400512: file test.cc, line 16.
(gdb) r
Starting program: /home/chenjunhan/Learning/gdb/test
Breakpoint 1, main () at test.cc:15
(gdb) continue
Continuing.
Breakpoint 2, main () at test.cc:16
```
3. `next`命令
next命令用于执行一行源程序代码，此行代码中的函数调用也一并执行；即“step over”，可简写为n。
4. `step`命令
step命令用于执行一行源程序代码，如果此行代码中有函数调用，则进入该函数；即“step into”，可简写为s。

## 2 gdb调试函数
- 列出可执行函数所有函数名称
```python
info|i functions
```
- 执行函数内容
```python
next|n	#不进入函数，直接执行函数并返回
step|s	#进入函数，执行函数体
call|print function	#任意位置直接执行函数
```
- 退出函数
```python
finish	#退出正在执行的函数，自动执行剩下的代码
return [expression]	#退出正在执行的函数，后面的代码不执行，可以通过expression修改函数返回值
```
- 函数堆栈帧
```python
info|i frame	#显示当前函数堆栈帧信息，包括指令寄存器的值，局部变量地址及值等信息。
backtrace|bt	#显示堆栈帧层次结构
frame n|address	#切换函数堆栈帧层次
up|down n	#向上/下选择n层函数堆栈帧
up-silently|down-silently n	#同上，不过不打印信息
```

## 3 gdb设置watchpoint
gdb可以使用`watch`命令设置观察点，也就是当一个变量值发生变化时，程序会停下来。例如：
```cpp
int a = 0;
void *pthread_func(void *args)
{
    while (1)
    {
        ++a;
        sleep(1);
    }
}
int main()
{
    pthread_t tid;
    pthread_create(&tid, NULL, pthread_func, NULL);
    sleep(1000);
    return 0;
}
```
使用`watch|wa`观察全局变量a:
```python
(gdb) file catch
Reading symbols from catch...done.
(gdb) break pthread_func
Breakpoint 1 at 0x400659: file catch.c, line 11.
(gdb) watch a
Hardware watchpoint 2: a
(gdb) r
Starting program: /home/chenjunhan/Learning/gdb/catch 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
[New Thread 0x7ffff77f6700 (LWP 6578)]
[Switching to Thread 0x7ffff77f6700 (LWP 6578)]

Breakpoint 1, pthread_func (args=0x0) at catch.c:11
11	        ++a;
(gdb) continue
Continuing.
Hardware watchpoint 2: a

Old value = 0
New value = 1
pthread_func (args=0x0) at catch.c:12
12	        sleep(1);
(gdb) continue
Continuing.

Breakpoint 1, pthread_func (args=0x0) at catch.c:11
11	        ++a;
```
- 设置观察点
```python
watch var	#为var设置观察点，在run之前
watch *(type*)(address)	#为地址address指向的变量设置观察点
```
- 列出所有观察点
```python
info watchpoints
```
- 断点控制命令
```python
disable|enable|delete n
```
- 为watch指定生效线程
```python
info threads	#列出所有threads编号等信息，thread_create之后
watch var thread t_num	#只有编号为t_num的thread修改了var值才会停下来
```
- 只读/读写观察点，只对硬件观察点才生效
```python
rwatch|rw var	#设置只读观察点每次访问var都会停下来
wwatch|aw var	#设置读写观察点，当发生读取或改变变量值的行为时，程序就会暂停住
```

## 4 gdb设置catchpoint
使用gdb调试程序时，可以用`tcatch`命令设置catchpoint只触发一次。例如：
```cpp
int main()
{
    pid_t pid;
    int i = 0;

    for (i = 0;i < 4; ++i)
    {
        if ((pid = fork()) < 0) //fork error
            exit(1);
        else if (pid == 0)  //child
            exit(0);
    }

    printf("Hello World\n");//parent
    return 0;
}
```
使用`tcatch fork`为fork设置catchpoint
```python
(gdb) tcatch fork
Catchpoint 1 (fork)
(gdb) r
Starting program: /home/chenjunhan/Learning/gdb/catch 

Temporary catchpoint 1 (forked process 6838), 0x00007ffff7ad5ee4 in __libc_fork
    () at ../nptl/sysdeps/unix/sysv/linux/x86_64/../fork.c:130
130	../nptl/sysdeps/unix/sysv/linux/x86_64/../fork.c: 没有那个文件或目录.
(gdb) c
Continuing.
Hello World
[Inferior 1 (process 6834) exited normally]
```

- 为fork/vfork/exec设置catchpoint
```python
catch fork
catch vfork
catch exec
```
- 为系统调用设置catchpoint
```python
catch syscall [syscall_name|syscall_num]	#例如catch syscall mmap
catch syscall	#为所有系统调用设置catchpoint
```

## 5 gdb高级打印技巧
#### 5.1 打印局部变量
- 使用`backtrace full`命令
```python
(gdb) bt full
#0  fun_a () at a.c:6
        a = 0
#1  0x000109b0 in fun_b () at a.c:12
        b = 1
#2  0x000109e4 in fun_c () at a.c:19
        c = 2
#3  0x00010a18 in fun_d () at a.c:26
        d = 3
#4  0x00010a4c in main () at a.c:33
        var = -1
```
- 使用`info locals`命令，打印当前函数局部变量
```python
(gdb) info locals
a = 0
```

#### 5.2 打印打印进程内存信息
```python
info proc mappings	#查看内存映像信息
info files	#更详细地输出进程的内存信息，包括引用的动态链接库等
info target	#功能同上
```

#### 5.3 打印变量类型
```cpp
typedef struct student_t{
	char *name;
	int age;
} student;

int main()
{
    student s;
    student *ps;

	return 0;
}
```
使用`wathis`和`ptype`命令调试，ptype可以列出详细的类型。
```python
(gdb) whatis s
type = student
(gdb) whatis ps
type = student *
(gdb) ptype s
type = struct student_t {
    char *name;
    int age;
}
```

#### 5.4 打印变量所在文件
```python
info variables var
```

## 6 gdb应用于多进程/线程
### 6.1 多进程调试
##### a.调试正在运行的进程
```python
#没启动gdb
ps -a | grep program	#查看进程id
gdb program pid	#调试
gdb --pid pid	#同上

#启动gdb
attach pid

#断开
detach
```
##### b.调试子进程
gdb调试默认追踪父进程，要追踪子进程，则执行下列命令：
```python
set follow-fork-mode child
```
##### c.同时调试父进程和子进程
gdb调试时，默认追踪一个进程，使用下列命令可以同时调试多个进程：
```python
set detach-on-fork off	#默认运行父进程，挂起其他进程
set schedule-multiple on	#父进程，子进程一起运行
```
之后gdb默认追踪父进程，其他进程被挂起，使用下列命令可以切换进程：
```python
inferior infno	#切换到子进程

info inferior	#打印所有进程信息
inferior n	#选择切换调试进程
```
### 6.2 多线程调试
##### a.显示线程信息
- `thread|_thread` 查看当前线程id
```python
(gdb) thread
[Current thread is 1 (Thread 0x7ffff7fcc740 (LWP 9494))]
(gdb) printf "current thread:%d\n",$_thread
current thread:1
```
- `info threads` 查看所有线程信息
```python
(gdb) info threads
Id   Target Id         Frame
3	Thread 0x7ffff6ff5700 (LWP 9627) "thread" 0x00007ffff78b7dfd in nanosleep () at ../sysdeps/unix/syscall-template.S:81
2	Thread 0x7ffff77f6700 (LWP 9626) "thread" 0x00007ffff78b7dfd in nanosleep () at ../sysdeps/unix/syscall-template.S:81
1	Thread 0x7ffff7fcc740 (LWP 9622) "thread" main () at thread.cc:21
```

##### b.切换线程
```python
thread id
```
##### c.只允许一个线程执行
gdb调试程序时，一旦程序断住，所有线程都会停止。当调试其中一个线程时，所有线程都会开始执行。如果想调试一个线程同时其他线程停止，可用以下命令：
```python
set scheduler-locking on
```
### 6.3 调试多个程序
```python
#添加一个新的调试程序。-copies指定执行多少份，默认1
add-inferior [-copies n] [-exec program]

#复制添加一个新的调试程序，infno表示调试进程编号，默认为当前inferior
clone-inferior [-copies -n] [infno]

info inferior	#列出所有调试的进程
maint info program-spaces	#列出所有调试的进程名称，inferior编号，进程id

inferior n	#切换进程
```

## 7 gdb分析core dump
### 7.1 core文件
程序由于各种异常或者bug导致在运行过程中异常退出或者中止，并且在满足一定条件下（这里为什么说需要满足一定的条件呢？下面会分析）会产生一个叫做core的文件。
通常情况下，core文件会包含了程序运行时的内存，寄存器状态，堆栈指针，内存管理信息还有各种函数调用堆栈信息等，我们可以理解为是程序工作当前状态存储生成第一个文件，许多的程序出错的时候都会产生一个core文件，通过工具分析这个文件，我们可以定位到程序异常退出的时候对应的堆栈调用等信息，找出问题所在并进行及时解决。
ubuntu默认不生成core文件，可用一下命令解决：
```python
ulimit -c unlimited
```
### 7.2 gdb显式生成core文件
```python
generate-core-file	#先run
gcore	#上面的简写
```
### 7.3 使用core文件进行调试
示例程序：
```cpp
int main()
{
    int *p = NULL;
    *p = 0;

    return 0;
}
```
上面程序当执行`*p = 0`时程序会crashed，产生core dump file，下面使用gdb进行调试：
```python
#shell环境加载core文件
gdb test corefile

#同上，gdb环境中
file test
core corefile

#gdb加载core file之后打印的信息
[New LWP 9672]
Core was generated by `./test'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0  0x000000000040054b in main () at test.c:8
8	    *p = 0;

#也可以使用where命令定位错误位置
(gdb) where
#0  0x000000000040054b in main () at test.c:8
```




[参考文献](https://github.com/hellogcc/100-gdb-tips/blob/master/src/index.md)



