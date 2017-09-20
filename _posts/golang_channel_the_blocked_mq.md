---
title: Go Channel原理--阻塞式消息队列
toc: true
comments: true
date: 2017-07-14 10:46:10
tags: channel
categories: Go
---

channel是Go语言在语言级别提供的goroutine间的通信方式。我们可以使用channel在两个或 多个goroutine之间传递消息。

channel是类型相关的。也就是说，一个channel只能传递一种类型的值，这个类型需要在声 明channel时指定。可以以将其认为是一种类 型安全的管道(pipe)。

<!--more-->

### 阻塞与非阻塞

channel内部实现的其实是一个阻塞式消息队列，每当有一个goroutine从channel中读取数据时，如果channel buffer中没有数据，那么读取操作会一直阻塞。同理，往channel中发送数据时，如果channel buffer是满的，那么发送操作也会阻塞。

```go
c1 := make(chan int)
val := <-c1	//block

c2 := make(chan int, 2)
c2 <- 0
c2 <- 1
c2 <- 3		//block
```



### 线程安全

在多线程并发编程中，多个goroutine同时读写同一个channel，channel采用加锁来避免race，从而实现了线程安全。

```go
run := func(c <-chan int) {
  	c <- 0
}

c := make(chan int)
go run(c)	//thread safe
go run(c)	//thread safe
```

channel在Go内部其实是下面的hchan类型，该结构体中，通过一个lock的锁对象来保护读写其他字段。

```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```

### 异常读写

#### 给一个nil-channel发送数据，将永远阻塞

```go
func writeNilChannel() {
	var c chan int
	c <- 1 //fatal error: all goroutines are asleep - deadlock!
}
```

#### 从一个nil-channel读取数据，将永远阻塞

```go
func readNilChannel() {
	var c chan int
	val := <-c //fatal error: all goroutines are asleep - deadlock!
	fmt.Println(val)
}
```

出现上面两种情况的原因是：

- channel 的 buffer 的大小不是类型声明的一部分，因此它必须是 channel 的值的一部分
- 如果 channel 未被初始化，它的 buffer 的大小将是0，也就是没有buffer
- 如果 channel 没有 buffer，一个发送将会被阻塞，直到另外一个 goroutine 为接收做好了准备
- 如果 channel 没有 buffer，一个接收将会被阻塞，直到另外一个 goroutine 发送了数据

#### 给一个closed-channel发送数据，引起panic

```go
func writeClosedChannel() {
	c := make(chan int)
	close(c)
	c <- 1 //panic: send on closed channel
}
```

#### 从一个closed-channel读取数据，立即返回数据类型的默认值

```go
func readClosedChannel() {
	c4Int := make(chan int)
	close(c4Int)
	vi := <-c4Int 	//0
	
  	c4Str := make(chan string)
  	close(c4Str)
  	vs := <-c4Str	//""
  
  	c4Ptr := make(chan *int)
  	close(c4Ptr)
  	vp := <-c4Ptr	//nil
}
```

#### 有效判断channel是否closed

```go
//for range
for v := range c {
  	//do something with v
}

//or if
if v, ok := <-c; ok {
  	//do something with v
}
```

