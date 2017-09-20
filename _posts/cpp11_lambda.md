---
title: lambda函数
toc: true
date: 2017-04-19 22:34:30
tags: lambda
categories: C++
---
Lambda表达书是C++11新增的一个特性，Lambda又称Lambda函数，是C++中的匿名函数。
<!--more-->
## Lambda函数形式
Lambda函数的基本形式是：
`[ capture ] ( params ) mutable exception attribute -> ret { body }`
`[ capture ] ( params ) -> ret { body }`
`[ capture] ( params ) { body }`
`[ capture] { body }`
- 第一个是Lambda表达式的完成形式，
- 第二个是const类型的Lambda表达式，不能修改[]列表捕获的变量
- 第三个是忽略返回类型的Lambda表达式，返回类型通过以下两种规则自动推导
 - 如果body含有return语句，则返回类型为return的值的类型
 - 如果body没有return语句，则返回类型为`void func(...)`的函数类型

**mutable**修饰符说明 lambda 表达式体内的代码可以修改被捕获的变量，并且可以访问被捕获对象的 non-const 方法。
**exception**说明 lambda 表达式是否抛出异常(noexcept)，以及抛出何种异常，类似于void f() throw(X, Y)。
**attribute**用来声明属性。
** capture **用来声明捕获列表，指定了可见域范围内Lambda表达式可见的外部变量列表，具体如下：
- ** [] **：不捕获外部变量
- ** [&] **：以引用类型捕获所有外部变量
- ** [=] **：以值捕获所有外部变量
- ** [a, &b] **：a以值方式捕获，相当于捕获副本，b以引用捕获
- ** [this] **：以值方式捕获this指针

## 一个简单的例子
```Cpp
#include <iostream>

using namespace std;

int main()
{
	auto func = [](){ cout << "Hello World!" << endl; };
	func();	//func自动推导类型为void foo()函数类型
	
	int a = 2;
	auto divA = [=] (int val) -> int{ return val / a; }	//返回类型int func(int)函数类型
	cout << "10 div a = " << divA(10) << endl;

	return 0;
}

/*
输出：
Hello World
10 div a = 5
*/
```

## Lambda函数的用处
Lambda函数一般用于以下情形：
如果代码里面存在大量的小函数，而这些函数一般只被调用一次。
又或者是提供一个接口，具体实现由用户自定义。
例如：
```cpp
class Something
{
public:
	template<typename CheckFunc>
	bool find(CheckFunc func)
	{
		for (auto beg = m_info.begin(); beg != m_info.end(); ++beg)
		{
			if (func(*beg))
				return true;
		}
		return false;
	}
private:
	std::vector<std::string> m_info;
};

//用户调用find接口时，check函数一般只作为参数传入时被调用，其他时候不需要使用，这个时候就可以使用Lambda表达式：
Something some;
//initialized
some.find( [](std::string str) { return str == "Hello"; } );
```
