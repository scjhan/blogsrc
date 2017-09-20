---
title: Redis基础
toc: true
comments: true
date: 2017-07-11 09:40:54
tags: redis
categories: Redis
---



### 1. key

#### 1.1基础操作

<!--more-->

```
# 删除一个key
del key

# 判断是否存在(0/1)
exists normal:key1

# 查找
keys pattern

# 判断类型
type normal:key1

# 设置存活时间
expire normal:key1 120 #秒
pexpire normal:key1 120 #毫秒

# 获取存活时间（不存在-2，未设置-1）
ttl normal:key1  # 秒
pttl normal:key1 #毫秒

# 删除存活时间设定
persist normal:key1
```

#### 1.2 序列化与反序列化

```
#返回一个序列化值"\x00\x0bhello redis\a\x00(O\t?\x9c\x8b\xba\xcd"
dump test:key

#序列化一个值到指定key
# ttl设置为0表示不设置，不加replace则反序列化到同一个键该值出错
restore test:key2 ttl "\x00\x0bhello redis\a\x00(O\t?\x9c\x8b\xba\xcd" [replace]
```



### 2. 数据结构

#### 2.1 字符串（String）

Redis中，最常见数据结构的就是字符串

```
set test:strings:key1 "Hello Redis"
set test:strings:key2 "Hello World"
get test:strings:key1
get test:strings:key2

# 上述等价于
mset test:strings:key1 "Hello Redis" test:strings:key2 "Hello World"
mget test:strings:key1 test:string:key2

# 获取字符串长度
strlen test:strings:key1

# 获取sub string
getrange test:strings:key1 6 10

# 追加
append test:strings:key1 "!"
```

#### 2.2 哈希表（Hash）

散列数据结构提供了一个额外的间接层：一个域（Field）

```
hset test:hash:key1 name "junhan"
hset test:hash:key1 age 21
hset test:hash:key1 phone "188*******6"

# 上述等价于
hmset test:hash:key1 name "junhan" age 21 phone "188*******6"

# 根据key field获取数据
hget test:hash:key1 name

# 获取多个
hmget test:hash:key1 name age phone

# 获取所有field value
hgetall test:hash:key1
```

#### 2.3 链表（List）

链表数据结构可以存储一组值，主要操作是添加一个值到第一个或者最后一个位置，以及通过索引来访问数据。

```
# 从左端逐个插入数据
lpush program:language c c++ java go

# 从右端逐个插入数据
rpush program:language python php

# 获取长度
llen program:language

# 根据索引获取
lindex program:language 0

# 弹出数据
lpop program:language
rpop program:language

# 数据截取(留下[s, e)范围，其他删除)
ltrim 0 4
```

#### 2.4 集合（Set）

集合用来存储唯一的值，添加相同的值则本次操作将会被忽略

```
sadd set:program:language1 c c++ java go
sadd set:program:language2 python php java

# 获取所有元素
smembers set:program:language1

# 判断是否存在
sismember set:program:language1 java

# 取交集
sinter set:program:language1 set:program:language2

# 取差集
sdiff set:program:language1 set:program:language2

# 取并集
sunion set:program:language1 set:program:language2

# 删除元素
srem set:program:language2 java
```

#### 2.5 有序集合（SortedSet）

sortedset是一种集合类数据结构，采用score来标记一个元素，并通过score来进行排序，这里的score相当于rank，它的取值可以是整数值或双精度浮点数。

```
zadd sortset:test:key1 1 google.com 2 microsoft.com 3 apple.com 4 amazon.com 10 baidu.com

# 递增列出数据
zrange sortset:test:key1 0 3

# 递减列出数据
zrevrange sortset:test:key1 0 3

# 通过score计数
zcount sortset:test:key1 2 4

# 获取score
zscore sortset:test:key1 google.com

# 获取rank（根据score排的结果）
zrank sortset:test:key1 baidu.com # 4

# 删除
zrem sortset:test:key1 baidu.com
```



### 3. 事务（Transaction）

#### 3.1 Redis事务原理

Redis是单线程运行的，所以能够保证每个命令的原子性。但在实际开发中可能需要原子地执行一组命令，这个时候可以用redis的事务，它保证：

- 事务中的命令顺序执行
- 事务中的命令如单个原子操作般被执行
- 事务中的命令要么都被执行，要么都不执行

```
multi
set test:tran:key1 "test"
get test:tran:key1
# ...
exec # 执行事务
discard # 放弃
```

#### 3.2 事务并发问题

虽然Redis是单线程的，但是在起多个redis-cli的时候，还是会出现并发中的一些问题，例如上面的代码，在get之后，set之前，`test:tran:key1`可能会被其他Redis进程修改，进而引起错误。对于这种情况，可以采用`watch`的监视。

```
watch test:tran:key1
get test:tran:key1
multi
set test:tran:key1 "new value"
exce

# 取消监视
unwatch test:tran:key1
```

如果在set之前有其他客户端修改了`test:tran:key1`，那么事务将会执行失败。
