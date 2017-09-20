---
title: 栈和队列及其基本操作
toc: true
date: 2017-04-18 13:14:12
tags: 
	- stack
	- queue
categories: 数据结构与算法
---
## 1.栈(stack)
如果一个线性结构只允许在序列末端进行操作，则称为栈，栈具有后进先出（Last in first out, LIFO）特点。  
按实现方式，栈分为顺序栈和链式栈两种。前者用一维数组实现，后者用链表实现。
<!--more-->
#### 1.1 顺序栈
每当新元素入栈，栈顶索引向后移动，出栈栈顶索引前移。这种实现比较方便，不用移动元素，但是数组必须预先确定大小，若数组空间满了，就必须分配更大的空间进行扩容。
```cpp
#define MAX_SIZE 1024
class stack{
public:
    stack() : m_top(-1), mp_elem(new int[MAX_SIZE];) {}
    ~stack() { delete[] mp_elem; }
    
    void pop() { --m_top; }
    int top() { return mp_elem[m_top]; }
    bool push(int val)
    {
        if (m_top == MAX_SIZE - 1) return false;
        mp_elem[++m_top] = val;
    }
    bool empty() { return m_top == 0; }
    void clear() { m_top = 0; }
private:
    int *mp_elem;
    int m_top;
};
```
#### 1.2链式栈
使用链表维护一个栈，链表头结点表示栈顶结点，每次入栈都新分配一个结点，插到头结点前面即可。
```cpp
struct stack_node {
    stack_node(int val) : value(val), next(nullptr) {}
    int value;
    struct stack_node *next;
};
class stack {
    stack() : head(nullptr) {}
    ~stack() { clear(); }
    
    void push(int val)
    {
        struct stack_node *pv = new struct stack_node(val);
        pv->next = head;
        head = pv;
    }
    int top() { return head->val; }
    void pop()
    {
        struct stack_node *p = head;
        head = head->next;
        delete p;
    }
    bool empty() { return !head; }
    void clear()
    {
        struct struct stack_node *p = head;
        while (head)
        {
            p = head->next;
            delete head;
            head = p;
        }
    }
private:
    struct stack_node *head;
};
```
#### 1.3应用举例——括号匹配问题
> **Problem Description**   
括号匹配序列允许()和[]，嵌套顺序任意。例如'([])()'符合， '([)]'不符合。  

```cpp
bool judge(const string &str)
{
    stack s;
    for (int i = 0;i < str.length(); ++i)
    {
        if (str[i] == '(' || str[i] == ')')
            s.push(str[i]);
        else if (s.empty()) return false;
        else if (str[i] == '(' && s.top() == ')' || str[i] == '[' && s.top() == ']')
            return true;
        else return false;
    }
    return false;
}
```
****
## 2.队列
队列和栈类似，不同的是，队列是先入先出(First in first out, FIFO)  
队列一般使用循环结构实现，避免空间不足。例如，对于一个使用一维数组实现的队列。  
#### 2.1顺序循环队列
```
//队列 {1, 2, 3, 4}
----------------------------------
| f 1  |  2  | 3  |  4  |  rear  |
----------------------------------
```
使用两个指针，front和rear分别指向队头和队尾后一个元素。每次插入元素，，元素出队，front指针向前移动。
- 插入入队时，rear指针向后移动，**rear = (rear + 1) % MAX_SIZE**，插入位置为rear
- 元素出队时，front指针向前移动，**front = (front - 1 + MAX_SIZE) % MAX_SIZE**。
- 判空条件：**front = (rear + 1) % MAX_SIZE**
- 判满条件：**front = rear**
```cpp
#define MAX_SIZE 1024
class queue {
public:
    queue() : m_front(0), m_rear(1) : pm_elem(new int[MAX_SIZE];) {}
    ~queue() { delete pm_elem; }
    
    bool empty() { return  m_front = (m_rear + 1) % MAX_SIZE; }
    bool push(int val)
    {
        if (m_front == m_rear) return false;
        pm_elem[rear] = val;
        m_rear = (m_rear + 1) % MAX_SIZE;
    }
    int front() { return pm_elem[front]; }
    int back() { return pm_elem[rear]; }
    bool pop()
    {
        if (empty()) return false;
        m_front = (m_front + 1) % MAX_SIZE;
    }
    
private:
    int *pm_elem;
    unsigned int mfront;
    unsigned int m_rear;
};
```
#### 2.2 链式队列
采用链表实现循环队列，和上面差不多，使用两个指针front和rear分别指向链表头结点和尾结点。
- 判空条件：**front == nullptr**

****

## 3.常见问题
#### 3.1 实现一个栈，要求实现出栈，入栈，Min返回最小值的操作的时间复杂度为o(1)
- 思路：要使这些操作的时间复杂度为o(1),则必须保证栈的每个元素只被遍历一次。求解时需要借助两个栈，一个入数据，一个入所遍历过数据的最小值，遍历结束后，放最小值的栈的栈顶元素即为所求的最小值。  
```cpp
#include <stack>
#include <assert.h>

template<typename T>
class stackmin{
    stackmin() {}
    void push(T val)
    {
        datastack.push(val);
        if (minstack.empty() || val < minstack.top())
            minstack.push(val);
        else minstack.push(minstack.top());    
    }
    void pop()
    {
        assert_not_empty();
        datastack.pop();
        minstack.pop();
    }
    int min() const
    {
        assert_not_empty();
        return minstack.top();
    }
private:
    stack<T> datastack;
    stack<T> minstack;
    void assert_not_empty()
    {
        assert(datastack.size() > 0 && minstack.size() > 0);
    }
};

```
#### 3.2 两个栈实现队列
- 思路：两个栈，一个入数据，一个出数据。对于入队栈，不论空不空，直接push；对于出队栈，若空则先把出队栈倒入入队栈，再出队。
```cpp
template<typename T>
class queue {
  queue() {};
  ~queue() {};
  
  void push(const T &t) { in_stack.push(t); }
  void pop()
  {
      if (out_stack.empty())
      {
          while (!in_stack.empty())
          {
              out_stack.push(in_stack.top());
              in_stack.pop();
          }
      }
  }
private:
    stack<T> in_stack;
    stack<T> out_stack;
};
```
#### 3.3两个队列实现栈
- 思路：一个队列作为数据队列，有数据入栈就入队，另一个作为辅助队列，每当出栈，将数据队列入到辅助队列，最后一个弹出。
```cpp
template <typename T>class CStack
{
public:
    CStack(void){};
    ~CStack(void){};
    void push(const T& node);
    T pop();
private:
    queue<T> queue1;
    queue<T> queue2;
    
};

//插入元素
template <typename T> void CStack<T>::push(const T& element)
{
    if(queue1.size()>0)//如果queue1不为空则往queue1中插入元素
        queue1.push(element);
    else if(queue2.size()>0)//如果queue2不为空则往queue2中插入元素
        queue2.push(element);
    else//如果两个队列都为空，则往queue1中插入元素
        queue1.push(element);
        
}

//删除元素
template <typename T> T CStack<T>::pop()
{
    if(queue1.size()==0)//如果queue1为空
    {
        while(queue2.size()>1)//保证queue2中有一个元素，将其余元素保存到queue1中
        {
            queue1.push(queue2.front());
            queue2.pop();
        }
        T& data=queue2.front();
        queue2.pop();
        return data;
    }
    else//如果queue2为空
    {
        while(queue1.size()>1)//保证queue2中有一个元素，将其余元素保存到queue1中
        {
            queue2.push(queue1.front());
            queue1.pop();
        }
        T& data=queue1.front();
        queue1.pop();
        return data;
    }

}
```
