---
title: 可变参数包与完美转发
toc: true
date: 2017-04-19 22:34:38
tags: 
	- template
	- variadic template
categories: C++
---
## 可变参数模板
可变参数模板(variadic template)是C++11新增的一项特性，使得模板参数可以任意化。一个基本的可变参数模板声明如下：
<!--more-->
```cpp
template<typename ...Element> class Tuple;

//使用
Tuple<int, double, string> tup;
```
其中Element被称为模板参数包(template type parameter pack)，参数包的解包采用的是模板特化的方法：
```cpp
template<typename Head, typename ...Tail>
class tuple<Head, Tail...> : private tuple<Tail...> {};

//特化版本
template<> class tuple<> {};
```

## 完美转发
假设有下面这样的函数，用来转发参数
```cpp
template<typename Arg>
void forwardArg(TYPE arg)
{
	foo(arg);
}
```
- arg是`Arg`类型，即值传递，那么arg可能在forwardArg里面被修改，导致传给foo的参数不一致
- arg是`Arg&`类型，即引用传递，那么forward又没办法接收右值类型
- arg是`Arg&`类型，并重载了const T&的方法，这个时候可以接收右值类型，但是对于多个参数，必须考虑多种情况，这个时候重载多个版本显然不能解决问题。

右值引用的引入解决了这个问题，在这种上下文时，它成为forwarding reference。例如下面情况:
```cpp
template <class T>
void func(T&& param);
```
其中，`T&&`不一定表示右值，如果它绑定的类型未知，那么结果也是未知的，可能是左值，也可能是右值。称其为**未定引用类型(universal reference)**，这种类型必须被初始化，并在初始化中决定其类型。

#### 1.引用折叠规则
> 由于存在T&&这种未定的引用类型，当它作为参数时，有可能被一个左值引用或右值引用的参数初始化，这是经过类型推导的T&&类型，相比右值引用(&&)会发生类型的变化，这种变化就称为引用折叠。
> ——《深入应用C++11-代码优化与工程级应用》,祁宇,P68

对于`C++`语言，不可以在源程序中直接对引用类型再施加引用。`T& &`将编译报错。C++11标准中仍然禁止上述显式对引用类型再施加引用，但如果在上下文环境中（包括模板实例化、typedef、auto类型推断等）如出现了对引用类型再施加引用，则施行[引用塌缩规则][1]（reference collapsing rule）
- `T& &`变为`T&`
- `T& &&`变为`T&`
- `T&& &`变为`T&`
- `T&& &&`变为`T&&`

#### 2.模板参数类型推导
对于函数模板`template<class T> void foo(T&& param);`，应用引用折叠规则分析如下：
- 实参是类型A的左值，则模板参数T的类型为`A&`，形参类型为`A&`
- 实参时类型A的右值，则模板参数T的类型为`A`，形参类型为`A&&`

#### 3.完美转发及其实现
完美转发（perfect forwarding）问题是指函数模板在向其他函数传递参数时该如何保留该参数的左右值属性的问题。也就是说函数模板在向其他函数传递自身形参时，如果相应实参是左值，它就应该被转发为左值；同样如果相应实参是右值，它就应该被转发为右值。
C++11提供了`std::forward (include <unility>)`来实现完美转发，具体操作如下：
```cpp
template <typename ...Args>
void forward_ref(Args&& ...args)
{
	foo(std::forward<Args>(args)...);
}
```
利用折叠规则，可以写出forward大致实现，如下：
```cpp
template<typename T>
T&& forward(typename std::remove_reference<T>::type& t)
{
	return static_cast<T&&>(t);
}
```
下面分析下完美转发的工作原理：
1. 如果`forward_ref`传入的是类型A的左值，根据折叠规则，Args被决议为A&类型，`forward<A&>`实例化之后返回值为`static_cast<A& &&>`也即`static_cast<A&>`，符合预期。
2. 如果`forward_ref`传入的是类型A的右值，Args被决议为A类型，`forward<A>`实例化后返回`static_cast<A&&>`也符合预期。

## 参考资料
[如何理解c++中的引用折叠?][2]
[C++11:深入理解右值引用，move语义和完美转发][2]
[C++11 图说VS2013下的引用叠加规则和模板参数类型推导规则][4]

  [1]:  https://zh.wikipedia.org/wiki/%E5%8F%B3%E5%80%BC%E5%BC%95%E7%94%A8#.E5.BC.95.E7.94.A8.E6.8A.98.E5.8F.A0.E8.A7.84.E5.88.99
  [2]: https://www.zhihu.com/question/40346748
  [3]: http://blog.csdn.net/booirror/article/details/45057689
  [4]: http://www.cnblogs.com/zpcdbky/p/4483479.html
