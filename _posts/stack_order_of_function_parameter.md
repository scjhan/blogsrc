---
title: 函数调用参数入栈顺序
toc: true
comments: true
date: 2017-08-23 22:13:37
tags: gdb
categories: 函数参数
---

之前一直以为函数的参数入栈顺序是从右向左，但事实上不同架构的系统实现是不同的。具体分析如下。

<!--more-->

### 32位系统

32位系统：参数从右到左入栈 

具体分析可看汇编程序，先给出测试代码

```c++
class MC {
public:
    int address(int a, int b) {
      return 0;
    }
};
 
int main()
{
    MC mc;
    mc.address(1, 2);

    return 0;
}
```

gdb列出汇编代码，如下

```shell
(gdb) disass
Dump of assembler code for function main():
   0x080484c8 <+0>:     lea    0x4(%esp),%ecx
   0x080484cc <+4>:     and    $0xfffffff0,%esp
   0x080484cf <+7>:     pushl  -0x4(%ecx)
   0x080484d2 <+10>:    push   %ebp
   0x080484d3 <+11>:    mov    %esp,%ebp
   0x080484d5 <+13>:    push   %ecx
   0x080484d6 <+14>:    sub    $0x14,%esp
=> 0x080484d9 <+17>:    sub    $0x4,%esp
   0x080484dc <+20>:    push   $0x2
   0x080484de <+22>:    push   $0x1
   0x080484e0 <+24>:    lea    -0x9(%ebp),%eax
   0x080484e3 <+27>:    push   %eax
   0x080484e4 <+28>:    call   0x80484fa <MC::address(int, int)>
   0x080484e9 <+33>:    add    $0x10,%esp
   0x080484ec <+36>:    mov    $0x0,%eax
   0x080484f1 <+41>:    mov    -0x4(%ebp),%ecx
   0x080484f4 <+44>:    leave
   0x080484f5 <+45>:    lea    -0x4(%ecx),%esp
   0x080484f8 <+48>:    ret
```

可以看到11-14行，汇编代码是先`push $0x2 `然后` push $0x1`，最后才`push %eax `，这个是this指针。也就是说参数入栈顺序为后面的参数先入栈。

### 64位系统

64位系统稍微有点不同，前六个参数先放在寄存器中，其他参数从后面开始入栈。

测试代码如下

```c++
class MC {
public:
    void foo(int a1, int a2, int a3, int a4, int a5, int a6, int a7) {
        return;
    }
};

int main()
{
    MC mc;
    mc.foo(1, 2, 3, 4, 5, 6, 7);

    return 0;
}
```

gdb汇编代码如下

```c++
(gdb) disass
Dump of assembler code for function main():
   0x00000000004005b0 <+0>:	push   %rbp
   0x00000000004005b1 <+1>:	mov    %rsp,%rbp
   0x00000000004005b4 <+4>:	sub    $0x20,%rsp
=> 0x00000000004005b8 <+8>:	lea    -0x1(%rbp),%rax
   0x00000000004005bc <+12>:	movl   $0x7,0x8(%rsp)
   0x00000000004005c4 <+20>:	movl   $0x6,(%rsp)
   0x00000000004005cb <+27>:	mov    $0x5,%r9d
   0x00000000004005d1 <+33>:	mov    $0x4,%r8d
   0x00000000004005d7 <+39>:	mov    $0x3,%ecx
   0x00000000004005dc <+44>:	mov    $0x2,%edx
   0x00000000004005e1 <+49>:	mov    $0x1,%esi
   0x00000000004005e6 <+54>:	mov    %rax,%rdi
   0x00000000004005e9 <+57>:	callq  0x4005f6 <MC::foo(int, int, int, int, int, int, int)>
   0x00000000004005ee <+62>:	mov    $0x0,%eax
   0x00000000004005f3 <+67>:	leaveq 
   0x00000000004005f4 <+68>:	retq
     
//进入foo函数后
Dump of assembler code for function MC::foo(int, int, int, int, int, int, int):
   0x00000000004005f6 <+0>:	push   %rbp
   0x00000000004005f7 <+1>:	mov    %rsp,%rbp
   0x00000000004005fa <+4>:	mov    %rdi,-0x8(%rbp)
   0x00000000004005fe <+8>:	mov    %esi,-0xc(%rbp)
   0x0000000000400601 <+11>:	mov    %edx,-0x10(%rbp)
   0x0000000000400604 <+14>:	mov    %ecx,-0x14(%rbp)
   0x0000000000400607 <+17>:	mov    %r8d,-0x18(%rbp)
   0x000000000040060b <+21>:	mov    %r9d,-0x1c(%rbp)
=> 0x000000000040060f <+25>:	nop
   0x0000000000400610 <+26>:	pop    %rbp
   0x0000000000400611 <+27>:	retq   
End of assembler dump.
```

- 7-14行，后面的参数7和6先入栈，然后前面1-5以及this指针按照从右向左的顺序依次保存在寄存器中。
- 23-29行，先将栈中的参数7和6压入，然后在按照1-6寄存器的顺序将寄存器中的参数压入。

### 64位系统16个寄存器含义

![](/uploads/function_args_order_registers-64bit.jpg)

