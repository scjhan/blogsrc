---
title: 全排列算法
toc: true
date: 2017-04-18 13:13:32
tags: algorithms
categories: 数据结构与算法
---
## 0 描述
给定一个字符串，求出所有的全排列。例如"abc"的全排列为"abc","acb","bac","bca","cab","cba"。  
<!--more-->
### 1 递归解法
首先考虑bac和cba的由来，这两个都是abc的a和后面的两个交换而来；同理bca和cab分别由bac和cba的b和后面一个交换而来；acb由abc的b和后面一个交换而来。  
因此，对于递归的话，就是不断与后面的数交换的过程。不过这里还要注意的一点是避免交换重复的字符，也就是遇到重读直接跳过。
```cpp
vector<string> Permutation(string str) 
{
    vector<string> ret;
    
    permu(ret, str, 0);
    std::sort(str.beg(), str.end());
    
    return ret;
}

void permu(vector<string> &vec, string &str, int beg)
{
    if(beg == str.size()-1)
    {
        vec.push_back(str); //到底，保存此次结果
        return;
    }
    
    for (int i = beg; i < str.size(); ++i)
    {
        if (i == beg || str[i] != str[beg])  //重复不交换
        {
            std::swap(str[beg], str[i]);  //交换
            permu(vec, str, beg+1);       //递归
            std::swap(str[beg], str[i]);  //复位
        }
    }
}
```

### 2 非递归解法
非递归算法参考了stl的next_permutation算法实现，原理看的不太懂。  
先看看求出一个排列的下一个最小排列的算法： 
例如1342，从后往前找，找到第一个递增对，如这里是34，称第一个数为替换数，其坐标称为替换点。然后往后找比替换数大的最小数（一定存在），交换两者。例如这里1342变成1432。然后将替换点后的字符串前后颠倒。即得到1423，这个数就是1324的下一个最小全排列（原理不太懂）。  
所以整个算法的流程大致如下：
- 先给出str的最小排列，直接sort即可。
- 每次求出当前排列的下一最小排列，直到最大排列
```cpp
void swap(char *a, char *b)
{
    char t = *a;
    *a = *b;
    *b = t;
}

void reverse(char *a, char *b)
{
    while (a < b)
        swap(a++, b--);
}

bool next_permutation(char a[])
{
    char *pend = a + strlen(a);
    if (a == pend) return false;
    char *p, *q, *pfind;
    --pend;
    p = pend;

    while (p != a)
    {
        q = p--;
        if (*p < *q) //升序对
        {
            pfind = pend;
            while (*pfind <= *p)
                --pfind;
            swap(pfind, p);
            reverse(q, pend);
            return true;
        }
    }
    reverse(p, pend); //最大排列数
    return false;  //表示到达最大
}
```

## 3 标准库的std::next_permutation和std::prev_permutation
```cpp
#include <algorithm>

template<class BidirectionalIterator>
bool next_permutation(
      BidirectionalIterator _First, 
      BidirectionalIterator _Last
);
template<class BidirectionalIterator, class BinaryPredicate>
bool next_permutation(
      BidirectionalIterator _First, 
      BidirectionalIterator _Last,
      BinaryPredicate _Comp
 );
```
使用例子如下：
```cpp
#include <algorithm>
#include <cstring>

int main()
{
    char a[] = "abc";
    int len = strlen(a);
    do
    {
        printf("%s\n", a);
    } while (std::next_permutation(a, a+len));
    
    return 0;
}
```
