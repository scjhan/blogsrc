---
title: 堆及其基本操作
toc: true
date: 2017-04-18 13:12:58
tags: 
	- bitree
	- heap
categories: 数据结构与算法
---
## 0 描述
堆是具有以下特性的完全二叉树，其所有非叶子结点均不大于（或不小于）其孩子结点。
- 大顶堆  
所有非叶子结点均不小于其孩子结点
- 小顶堆  
所有非叶子结点均不大于其孩子结点  
<!--more-->
![](http://i2.buimg.com/567571/2194243e60b1e323.jpg)  
堆一般使用线性结构存储。
```cpp
class Heap
{
public:
//...
private:
    int *m_data;
    int m_size;
    int m_length;
    bool (*cmp)(int, int);
};
```

### 1 堆的筛选操作
堆的筛选作用是指定对根结点的子树进行堆特性的维护过程，并且假设子树满足堆特性。  
![](http://i2.buimg.com/567571/3d791bc289a8ac9c.jpg)  
```cpp
void shiftDown(int pos)
{
    int c, rc;
    while(pos < m_length / 2)
    {
        c = pos * 2 + 1;//lchild
        rc = pos * 2 + 2;//rchild

        if (rc < m_length && cmp(m_data[rc], m_data[c])) c = rc;

        if (cmp(m_data[pos], m_data[c])) return;

        std::swap(m_data[pos], m_data[c]);
        pos = c;//向下调整
    }
}
```
分析：  
对于深度为k的完全二叉树，做一个筛选最多需要比较2(k-1)次。n结点完全二叉树最大深度logn + 1，所以筛选复杂度为O(logn)。

## 2 堆的插入操作
插入时，先将元素插入到堆尾，这时，堆尾结点的双亲结点可能不满足堆特性，则堆尾与双亲交换，然后查看双亲结点是否符合堆特性，重复上述过程，直到符合堆特性或者到达根结点。  
![](http://i2.buimg.com/567571/48769d577dc1e584.jpg)  
```cpp
bool insert(int val)
{
    if (m_length == m_size) return false;

    m_data[m_length] = val;
    int curr = m_length++;
    while (curr != 0)
    {
        int parent = (curr-1) / 2;

        if (cmp(m_data[parent], m_data[curr])) break;

        std::swap(m_data[parent], m_data[curr]);
        curr = parent; //向上调整
    }
    return true;
}
```  
分析：  
插入最多判断logn次，时间复杂度为O(logn)。

## 3 堆的删除操作  
堆只能删除根结点，删除过程如下：
- 将尾结点复制到根结点，堆长度-1
- 对根结点进行筛选  
```cpp
bool pop()
{
    if (m_length == 0) return false;

    std::swap(m_data[0], m_data[--m_length]);
    if (m_length > 0) shiftDown(0);

    return true;
}
```

## 4 建堆操作
- 单个结点的完全二叉树满足堆特性
- 叶子结点满足堆特性  
由上，只需要按结点编号{n/2, n/2-1, ... , 0}的次序对结点进行筛选即可。
```cpp
Heap *create(int *a, int n, 
        bool (*cmp)(int, int) = less_than)
{
    if (a == nullptr || n < 0 || cmp == nullptr)
        return nullptr;
        
    int size = std::max(DEFAULT_SIZE, n);
    Heap *pheap = new Heap(size, cmp);
    pheap->m_length = size;
    for (int i = 0;i < n; ++i)
        pheap->m_data[i] = a[i];
    
    for (int i = n/2 - 1; i >= 0; --i)
        shiftDown(i);

    return pheap;
}
```
分析：  
建堆操作比较次数不超过4n，时间复杂度O(n)

## 5 全部代码
```cpp
#ifndef HEAP_H__
#define HEAP_H__

#include <algorithm>
#include <iostream>

static bool less_equal(int lhs, int rhs) { return lhs <= rhs; }
static bool greater_equal(int lhs, int rhs) { return lhs >= rhs; }

#define DEFAULT_SIZE 128

class Heap
{
public:
    Heap(int size = DEFAULT_SIZE, bool (*compare)(int, int) = less_equal) :
            m_data(new int[size]), m_size(size),
            m_length(0), cmp(compare) {}
    ~Heap() { delete[] m_data; }

    void shiftDown(int pos)
    {
        int c, rc;
        while(pos < m_length / 2)
        {
            c = pos * 2 + 1;//lchild
            rc = pos * 2 + 2;//rchild

            if (rc < m_length && cmp(m_data[rc], m_data[c])) c = rc;

            if (cmp(m_data[pos], m_data[c])) return;

            std::swap(m_data[pos], m_data[c]);
            pos = c;//向下调整
        }
    }

    bool insert(int val)
    {
        if (m_length == m_size) return false;

        m_data[m_length] = val;
        int curr = m_length++;
        while (curr != 0)
        {
            int parent = (curr-1) / 2;

            if (cmp(m_data[parent], m_data[curr])) break;

            std::swap(m_data[parent], m_data[curr]);
            curr = parent; //向上调整
        }
        return true;
    }

    bool pop()
    {
        if (m_length == 0) return false;

        std::swap(m_data[0], m_data[--m_length]);
        if (m_length > 0) shiftDown(0);
        return true;
    }

    Heap *create(int *a, int n, 
                bool (*cmp)(int, int) = less_than)
    {
        if (a == nullptr || n < 0 || cmp == nullptr)
            return nullptr;
        
        int size = std::max(DEFAULT_SIZE, n);
        Heap *pheap = new Heap(size, cmp);
        pheap->m_length = size;
        for (int i = 0;i < n; ++i)
            pheap->m_data[i] = a[i];
    
        for (int i = n/2 - 1; i >= 0; --i)
            shiftDown(i);

        return pheap;
    }

    //debug
    friend std::ostream& operator<<(std::ostream &os, const Heap &heap)
    {
        for (int i = 0;i < heap.m_length; ++i)
            os << heap.m_data[i] << " ";
        return os;
    }
private:
    int *m_data;
    int m_size;
    int m_length;
    bool (*cmp)(int, int);
};

#endif
```
