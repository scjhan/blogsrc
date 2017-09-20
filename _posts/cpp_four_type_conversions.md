---
title: C++四种类型转换
toc: true
date: 2017-04-19 22:34:59
tags: 类型转换
categories: C++
---
C++具有以下四种类型转换符：
- **static_cast**
- **const_cast**
- **dynamic_cast**
- **reinterpret_cast**
<!--more-->
具体用法一样，都是如下形式：
```cpp
xxx_cast<type>(expression)
```
### 1. static_cast
该运算符将expression转换成type类型，没有动态类型检查，不是安全的。而且无法移除原对象的**const**、**volital**等属性。它主要用于一下几种情况
- 用于内置数据类型（int,double等）的转换，不保证安全性
- 用于基类和派生类之间引用或者指针之间的转换
  - 向上转换（派生类转基类）是安全的
  - 向下转换（基类转派生类）没有动态类型检查，不安全
- 空指针转换成目标类型空指针
- 任何类型的expression转成void类型

```cpp
int f2i = static_cast<int>(3.14f);	//float->int

Base *pb = new Base();
Derive *pd = new Derive();
Base *pd2b = static_cast<Base*>(pd);		//安全
Derive *pb2d = static_cast<Derive*>(pb);	//不安全

Base b;
Derive d;
Base &rd2b = static_cast<Base&>(d);		//安全
Derive &rb2d = static_cast<Derive&>(b);	//不安全
```

### 2. const_cast
const_cast用来去掉const属性，主要有3种情况：
- 常量指针转换为非常量指针，仍然指向原有对象
- 常量引用转换为非常量引用，仍然绑定原有对象
- 常量对象转换为非常量对象

### 3. dynamic_cast
dynamic_cast的转换类型必须是引用或者指针。一般用于派生关系的上下转换，相比static_cast多了类型检查，即：
- 向下转换（派生类转基类），同**static_cast**
- 向上转换(基类转派生类)，转换失败，返回**nullptr**

```cpp
void func(Base *pb)
{
    Derive *d = dynamic_cast<Derive*>(pb);
    if (d == nullptr) cout << "A Base Pointer" << endl;
    else cout << "A Derive Pointer" << endl;
}

Base *pb = new Base();
Base *pd = new Derive();
func(pb);	//A Base Pointer
func(pd);	//A Derive Pointer
```

### 4. reinterpret_cast
**reinterpret_cast**运算符是用来处理无关类型之间的转换；它会产生一个新的值，这个值会有与原始参数（expressoin）有完全相同的比特位。但是：
> 错误的使用reinterpret_cast很容易导致程序的不安全，只有将转换后的类型值转换回到其原始类型，这样才是正确使用reinterpret_cast方式。

例如一个辅助hash函数：
```cpp
// Returns a hash code based on an address
unsigned short Hash( void *p ) {
	unsigned int val = reinterpret_cast<unsigned int>( p );
	return ( unsigned short )( val ^ (val >> 16));
}
```


