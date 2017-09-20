---
title: 右值引用与move语义
toc: true
date: 2017-04-19 22:34:50
tags:
	- rvalue
	- move
categories: C++
---
## 右值
C语言中，左值(left value, lvalue)只出现在赋值符左边的量，右值(right value, rvalue)是出现在赋值符右边的量。在`C++`中，右值的定义稍微不同，每一个表达式都会产生一个左值或者右值，所以表达式也称左值表达式或右值表达式。
- 对于基础类型，右值不可修改，也不可被const,volatile修饰
- 对于自定义类型，右值可以被成员函数修改
<!--more-->
```cpp
class Foo
{
public:
    Foo(int i) : m_i(i) {}
    void setI(int i) { m_i = i; }
private:
    int m_i;
};

Foo getFoo()	//返回右值，具有临时性
{
    return Foo(0);
}

int main()
{
    getFoo().setI(2);	//调用右值的成员函数，可以修改
    Foo *f = &getFoo();	//error: taking address of temporary

    return 0;
}
```
**非常量右值引用**
只能绑定到非常量右值，不能绑定到非常量左值、常量左值和常量右值。
**常量右值引用**
可以绑定到非常量右值和常量右值，不能绑定到非常量左值和常量左值

## move语义
考虑下面的一小段代码
```cpp
vector<int> getVector()
{
	vector<int> vec;
	return vec;
}
vector<int> v = getVector();
```
上述代码在没有优化的情况下，要调用三次构造函数，分别时构造vec，vec拷贝构造到返回值，返回值赋值构造到v。这样子性能特别差。如果确定某个值是一个非常量右值（或者是一个以后不会再使用的左值），则我们在进行临时对象的拷贝时，可以不用拷贝实际的数据，而只是“窃取”指向实际数据的指针。
例如上面的vec，在拷贝构造返回值时，它是一个以后绝对不会使用的左值，所以可以直接把实际数据指针赋给返回值，同理返回值赋值构造v时，返回值是一个非常量右值，也可以直接把实际数据指针赋给v。这样子实际数据拷贝就只有一次，大大提高了效率。
C++11提供了std::move操作用于上述情形，考虑下面的代码：
```cpp
vector<int> v{1, 2};
v = getVector();
```
第2句代码要做的工作有：
①销毁v原先的内存
②将getVector返回值内容拷贝一份给v
③销毁返回值的内存
如果vector定义了move赋值运算符:
```cpp
template<class T>
vector<T>& vector<T>::operator=(vector<T> &&rhs);
```
那么上述代码就会调用move赋值操作符，将返回值的内存指针赋给v。
move不仅适用于右值，可以以用于左值，std::move提供了将左值转成右值的功能。例如使用std::move实现swap：
```cpp
template <class T>
void swap(T &lhs, T &rhs)
{
	T tmp(std::move(lhs));
	lhs = std::move(rhs);
	rhs = std::move(tmp);
}
```
