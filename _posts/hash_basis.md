---
title: 哈希表及其基本操作
toc: true
date: 2017-04-18 13:14:27
tags: hash
categories: 数据结构与算法
---
## 0 哈希表概述
- 哈希表  
散列表（Hash table，也叫哈希表），是根据关键码值(Key value)而直接进行访问的数据结构。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。
<!--more-->
- 哈希函数
把关键字集合K映像到一个有限的连续地址集D的映射关系H表示为：  
**H(key):K->D,key∈K**  
映射关系H称为哈希函数。

## 1 哈希函数构造法
### 1.1 直接定址法
```cpp
int hash(int key)
{
    return a * key + b; //a为缩放系数，b为平移系数
}
```
优点：简单无冲突  
缺点： 空间浪费  
### 1.2 保留余数法
```cpp
int hash(int key)
{
    return key % p; //p取不大于地址长度m的最大素数
}
```
### 1.3数字分析法、折叠法、平方取中法

## 2 避免冲突
### 2.1开放定址法
- 线性探测法
```cpp
Hi = (H(key) + i) % m   //1 ≤ i ≤ m-1
```
优点：算法简单  
缺点：容易产生堆聚现象
- 二次探测法
```
di = 1, -1, 4, -4, 9, -9 ...
Hi = (H(key) + di) % m  ////1 ≤ i ≤ m-1
```
## 2.2链地址法
将关键西为同义词的记录链接在同一单链表中。
```
---------------------------------
|   |   |   |   |   |   |   |   |
---------------------------------
  |           |
-----       -----
|   |       |   |
-----       -----
```
简单实现
```cpp
typedef struct Node {
    RcdType r;
    struct Node *next;
} Node;
class hash_table


typedef struct{
    Node *rcd;
    int size;
    int count;
    int (*hash)(KeyType key, int hashSize);
} hash_table;

//初始化
void create_hash_table(hash_table &table, int size, int (*hash)(KeyType, int)
{
    table.rcd = (Node*)malloc(sizeof(Node) * size);
    table.size = size;
    table.hash = hash;
}

//查找
Node *search_hash_table(hash_table &tab, int key)
{
    int index = tab.hash(key, tab.size);
    for (Node *p = &tab.rcd[index], p != NULL; p = p->next)
    {
        if (p->r->key == key)
            return p;
    }
    return NULL;
}

//插入数据
bool insert_hash_table(hash_table &tab, RcdType r)
{
    if (search_hash_table(tab, r->key) == NULL)
    {
        int index = tab.hash(r.key, table.size);
        Node *p = (Node)malloc(sizeof(Node));
        p->r = r;
        p->next = tab.rcd[p];   //插入表头
        tab.rcd[p] = p;
        ++tab.count;
        return true;
    }
    return false;
}
```
## 3.C++11哈希表——std::unordered_map
`std::unordered_map`是C++新增的一个哈希表类型，和std::map类似，采用key-value结构，通过key快速索引到value。  
### 3.1实现机制
unordered_map内部实现了一个链式哈希表，即一个大bucket vecto挂着链表来解决冲突。  
![](http://i1.piimg.com/567571/bb10941041e7821e.jpg)  
1. hash_map其插入过程是：
- 得到key
- 通过hash函数得到hash值
- 得到桶号(一般都为hash值对桶数求模)
- 存放key和value在桶内。
2. 其取值过程是:
得到key
- 通过hash函数得到hash值
- 得到桶号(一般都为hash值对桶数求模)
- 比较桶的内部元素是否与key相等，若都不相等，则没有找到。
- 取出相等的记录的value。

### 3.2 部分源码
1. `unordered_map`在`bits/unordered_map.h`头文件中定义，对哈希表的所有操作都是对成员_M_h进行操作。
```cpp
template<...>
class unordered_map {
    //...
    _Hashtable _M_h;
};
```
2. _M_h是_Hashtable类型，定义在`bits/hashtable.h`头文件中。
>  Each _Hashtable data structure has:  
>   
>  - _Bucket[]       _M_buckets  
>  - _Hash_node_base _M_bbegin  
>  - size_type       _M_bucket_count  
>  - size_type       _M_element_count  

源码如下：
```cpp
template<...>
class _Hashtable {
    //...
private:
    __bucket_type*		_M_buckets;
    size_type			_M_bucket_count;
    __before_begin		_M_bbegin;
    size_type			_M_element_count;
    _RehashPolicy		_M_rehash_policy;
};
```

### 3.3 unordered_map用法
1. 一个简单的使用——key类型为内置类型
```cpp
#include <iostream>
#include <unordered_map>
#include <string>

int main()
{
    std::unordered_map<int, std::string> map;
    map.insert(std::make_pair(1, "one"));
    map.insert(std::make_pair(2, "tow"));
    map.insert(std::make_pair(3, "three"));

    auto iter = map.find(3);
    if (iter != map.end())
        std::cout << "Found:" +  iter->second;
    else std::cout << "Not found" << std::endl;

    return 0;
}
```
2. 高级用法——key类型为自定义类型
使用自定义类型时，需要定义hash_value函数并且重载operator==。
```cpp
struct Person  
{
public:
    //hash_value
    friend std::size_t hash_value(const Person &p)
    {
        std::size_t seed = 0;  
        std::hash_combine(seed, std::hash_value(p.name));  //combine
        std::hash_combine(seed, std::hash_value(p.age));  
        return seed;  
    }
    
    friend std::ostream& (std::ostream &os, const Person &p)
    {
        os << p.m_name << " " << p.m_age;
        return os;
    }

    Person(std::string name, int age) : m_name(name), m_age(age) {}
    
    //重载operator==
    bool operator==(const Person& p) const  
    {  
        return m_name == p.m_name && m_age == p.m_age;  
    }
private:
    std::string m_name;  
    int m_age; 
};

int main()  
{  
    typedef std::unordered_map<person,int> umap;  
    umap m;  
    Person p1("Tom1",20);  
    Person p2("Tom2",22);  
    Person p3("Tom3",22);  
    Person p4("Tom4",23);  
    Person p5("Tom5",24);  
    m.insert(umap::value_type(p3, 100));  
    m.insert(umap::value_type(p4, 100));  
    m.insert(umap::value_type(p5, 100));  
    m.insert(umap::value_type(p1, 100));  
    m.insert(umap::value_type(p2, 100));  
      
    for(umap::iterator iter = m.begin(); iter != m.end(); iter++)  
    {  
        std::cout << iter->first;
    }  
      
    return 0;  
} 
```

### 3.4 优缺点与map的比较
- map内部使用红黑树来维护，该结构具有自动排序的功能，因此map内部的所有元素都是有序的。红黑树的每一个节点都代表着map的一个元素，因此，对于map进行的查找，删除，添加等一系列的操作都相当于是对红黑树进行这样的操作，故红黑树的效率决定了map的效率。插入、查找复杂度一般为O(logn)。
- unordered_map，顾名思义，是一个无序map，内部实现了哈希表，因此插入、查找效率一般为O(1)
- 内存占用方面，map内存占用略低，unordered_map内存占用略高,而且是线性成比例的。

## 4 哈希表经典面试题
> 给定a、b两个文件，各存放50亿个url，每个url各占64字节，内存限制是4G，让你找出a、b文件共同的url？  

答案：可以估计每个文件安的大小为5G×64=320G，远远大于内存限制的4G。所以不可能将其完全加载到内存中处理。考虑采取分而治之的方法。
1. 遍历文件a，对每个url求取hash(url)%1000，然后根据所取得的值将url分别存储到1000个小文件（记为a0,a1,...,a999）中。这样每个小文件的大约为300M。 
2. 遍历文件b，采取和a相同的方式将url分别存储到1000小文件（记为b0,b1,...,b999）。这样处理后，所有可能相同的url都在对应的小文件（a0vsb0,a1vsb1,...,a999vsb999）中，不对应的小文件不可能有相同的url。然后我们只要求出1000对小文件中相同的url即可。
3. 求每对小文件中相同的url时，可以把其中一个小文件的url存储到hash_set中。然后遍历另一个小文件的每个url，看其是否在刚才构建的hash_set中，如果是，那么就是共同的url，存到文件里面就可以了。
