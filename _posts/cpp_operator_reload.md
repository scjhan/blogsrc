---
title: C++操作符重载
toc: true
comments: true
date: 2017-08-03 22:45:29
tags: C++
categories: operator
---

### TODO

- [ ] operator++ & operator--
- [ ] operator&
- [ ] operator*
- [ ] operator->
- [ ] operator=
- [ ] operator[]
- [ ] ​

<!--more-->

#### type operator()(args)

重载对象地`()`操作符，参数为args，一般用于仿函数。例如：

```c++
template<typename T>
class Less{
public:
	bool operator()(const T &lhs, const T &rhs) {
    	return lhs < rhs;
	}
}

int main()
{
	if (Less(1, 2)) {
    	//...
	}

	return 0;
}
```

#### operator type()

重载隐式转换操作，转换为type类型，例如：

```c++
class Bool{
public:
    operator bool(){
        return m_b;
    }
private:
    bool m_b;
};

int main()
{
    Bool b(true);
    if (b) {
        cout << "Bool::true" << endl;
    } else {
        cout << "Bool::false" << endl;
    }

    return 0;
}
```

