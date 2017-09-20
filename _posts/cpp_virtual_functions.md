---
title: C++虚函数表原理
toc: true
date: 2017-04-16 13:37:49
tags: 虚函数
categories: C++
---
## 前言
C++中的虚函数的作用主要是实现了多态的机制。关于多态，简而言之就是用父类型别的指针指向其子类的实例，然后通过父类的指针调用实际子类的成员函数。这种技术可以让父类的指针有“多种形态”，这是一种泛型技术。所谓泛型技术，说白了就是试图使用不变的代码来实现可变的算法。比如：模板技术，RTTI技术，虚函数技术，要么是试图做到在编译时决议，要么试图做到运行时决议。
<!--more-->
## 虚函数表
C++中，虚函数（Virtual Function）是通过一张虚函数表（Virtual Table）来实现的。简称为V-Table。在这个表中，主是要一个类的虚函数的地址表，这张表解决了继承、覆盖的问题，保证其容真实反应实际的函数。这样，在有虚函数的类的实例中这个表被分配在了这个实例的内存中，所以，当我们用父类的指针来操作一个子类的时候，这张虚函数表就显得由为重要了，它就像一个地图一样，指明了实际所应该调用的函数。
#### 1 虚函数表示例
C++的编译器保证虚函数表的指针存在于对象实例中最前面的位置（这是为了保证取到虚函数表的有最高的性能——如果有多层继承或是多重继承的情况下）。
例如有个如下的类：
```c++
class Base
{
public:
    virtual void foo()
    {
        cout << "Base::foo" << endl;
    }
    virtual void bar()
    {
        cout << "Base::bar" << endl;
    }
};
```
按照上面的说法，可以通过如下方式获取Base实例化对象的虚函数表：
```c++
Base b;
cout << "虚函数表地址： " << (long*)(&b) << endl;
cout << "第一个虚函数地址: " << (long*)*(long*)(&b) << endl;
```
输出结果为：
虚函数表地址： 0x7ffcc0c04c80
第一个虚函数地址: 0x400c20
通过上面分析，其实就可以通过访问虚函数表直接访问虚函数：
```c++
typedef void(*pFunc)();

long *vptr = (long*)(&b);
long *pVTable = (long*)(*vptr);
pFunc baseFoo = (pFunc)*(pVTable);
pFunc baseBar = (pFunc)*(pVTable+1);

baseFoo();	//Base::foo
baseBar();	//Base::bar
```
#### 2 虚函数表结构
下图展示了C++虚函数表的具体结构：
![虚函数表结构](http://img.blog.csdn.net/20170416124523825?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvU3RyYW5nZXJfQ0pIYW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
如图所示，可知：
- 对象实例保存了一个虚函数表地址指针vptr，指向单元存放的是虚函数表的真实地址，可以用如下代码表示:
  - `long *vptr = (long*)(&b);`
  - `long *pVTable = (long*)(*vptr);`
- 虚函数表是一个连续的地址空间，每一项存储一个虚函数的地址，按照函数声明顺序排列，可以用如下代码表示：
  - `pFunc base_foo = (pFunc)*(pVTable);`
  - `pFunc base_bar = (pFunc)*(pVTable+1);`
- 若对象有n个虚函数，实际上虚函数表有n+1项，其中*vptr指出的是第一项的地址，也就是第一个虚函数。第0项一般用作保存对象的**type_info**信息，用于保存该对象的实际类型。具体可用于dynamic_cast等，可用如下代码表示：
  - `const type_info *pti = (const type_info *)*(pVTable - 1)`

## 虚函数表实现多态
#### 1 单继承的情况
考虑如下代码：
```c++
class Derive : public Base
{
public:
    void foo()
    {
        cout << "Derive::foo" << endl;
    }
    virtual void foo1()
    {
        cout << "Derive::foo1" << endl;
    }
    virtual void bar1()
    {
        cout << "Derieve::bar1" << endl;
    }
};
```
对于一个Base类型的对象b和一个Derive类型的对象d，他们的虚函数表分别是：
```cpp
b-vtable:
	| type_info | Base::foo | Base::bar |
d-vtable:
	| type_info | Derive::foo | Base::bar | Deriev:foo1 | Derive::bar1 |
```
可以看到Derive重写了foo虚函数之后，虚函数表的相应位置被替换为Derive::foo，这个时候访问foo函数实际就访问了Derive::foo而不是Base::foo。例如：
```c++
Base *pb = new Derive();
pb->foo();	//Derive::foo
```
#### 2 多继承的情况
```c++
class Base1
{
public:
	virtual void base1_foo() { cout << "Base1::base1_foo() << endl; }
	virtual void base1_bar() { cout << "Base1::base1_bar() << endl; }
}
class Base2
{
public:
	virtual void base2_foo() { cout << "Base2::base1_foo() << endl; }
	virtual void base2_bar() { cout << "Base2::base2_bar() << endl; }
}

class Derive : public Base1, public Base2
{
public:
	void base1_foo() { cout << "Derive::base1_foo() << endl; }
	void base2_foo() { cout << "Derive::base2_foo() << endl; }
}
```
则Derive对象实际含有的是**2个vptr指针**，也即含有两个虚函数表，顺序按照继承顺序：
```c++
vptr1 --> vtable1：
	| type_info | Derive::base1_foo | Base::base1_bar |
vptr2 --> vtable2：
	| type_info | Derive::base2_foo | Base::base2_bar |
```











