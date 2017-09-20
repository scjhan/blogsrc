---
title: C++ this指针
toc: true
comments: true
date: 2017-08-23 10:42:47
tags: this
categories: c++
---

C++在创建对象时，系统会分配给每个对象一个独立的存储空间。而在类的内部，是通过隐式传递`this`来访问当前对象自己本身，也就是说只能在成员函数中访问到this指针。下面通过示例代码和gdb调试来探究this。

<!--more-->

### this指针不是类成员

```c++
#include <iostream>

using namespace std;

class MC{
public:
    MC() : val(1) {}
    void set_val(int v) {
        this->val = v;
    }
  	long address() { return &this; }
private:
    int val;
};

int main()
{
    MC mc;
    mc.set_val(100);

    return 0;
}
```

上述代码会输出4，即sizeof(val)的值，这说明this指针并不是类的成员。

### this指针是指针常量

```c++
class MC {
	long address() { return &this; }
};
```

通过上面可以得知，this指针实际上不是类的成员，于是尝试在打印出this指针的地址。然而上面的代码实际上会编译出错的，错误信息为：

> 单目‘&’的操作数必须是左值
>
> return &this;

**原因暂时不明**

在gdb查看this指针类型，可以看出，this指针实际上是个`Type *const`

```c++
(gdb) whatis this
type = MC * const
```

### this指针在成员函数通过参数隐式传入

上面说了this指针不是类成员，而是隐式传入给成员函数的。实际上，类对象在调用成员函数时，编译器将其作为第一个参数隐式传给成员函数的，也就是说，this指针实际上是调用时传递给成员函数的一个形参。

1. gdb step进入成员函数，可以看到函数传递的参数列表第一个就是this，同时也可以看到this指针的内容，也就是当前对象的首地址。

   ```
   MC::set_val (this=0x7fffffffe400, v=100) at thisgdb.cc:9
   9	        this->val = v;
   ```

2. 用gdb `info frame`查看栈帧如下，同样的也可以看到参数列表第一个就是this。

   ```
   (gdb) info frame
   Stack level 0, frame at 0x7fffffffe400:
    rip = 0x40077d in MC::set_val (thisgdb.cc:9); saved rip 0x400705
    called by frame at 0x7fffffffe420
    source language c++.
    Arglist at 0x7fffffffe3f0, args: this=0x7fffffffe400, v=100
    Locals at 0x7fffffffe3f0, Previous frame's sp is 0x7fffffffe400
    Saved registers:
     rbp at 0x7fffffffe3f0, rip at 0x7fffffffe3f8
   ```