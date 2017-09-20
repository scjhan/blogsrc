---
title: '以boost::function和Boost::bind取代虚函数'
toc: true
date: 2017-04-18 13:46:47
tags: 
	- function
	- bind
categories: 转载
---
这是一篇比较情绪化的blog，中心思想是“继承就像一条贼船，上去就下不来了”，而借助boost::function和boost::bind，大多数情况下，你都不用上贼船。

boost::function和boost::bind已经纳入了std::tr1，这或许是C++0x最值得期待的功能，它将彻底改变C++库的设计方式，以及应用程序的编写方式。

Scott Meyers的Effective C++ 3rd ed.第35条款提到了以boost::function和boost:bind取代虚函数的做法，这里谈谈我自己使用的感受。
<!--more-->

## 基本用途
boost::function就像C#里的delegate，可以指向任何函数，包括成员函数。当用bind把某个成员函数绑到某个对象上时，我们得到了一个closure（闭包）。例如：
```cpp
class Foo
{
 public:
  void methodA();
  void methodInt(int a);
};
class Bar
{
 public:
  void methodB();
};

boost::function<void()> f1; // 无参数，无返回值

Foo foo;
f1 = boost::bind(&Foo::methodA, &foo);
f1(); // 调用 foo.methodA();
Bar bar;
f1 = boost::bind(&Bar::methodB, &bar);
f1(); // 调用 bar.methodB();

f1 = boost::bind(&Foo::methodInt, &foo, 42);
f1(); // 调用 foo.methodInt(42);

boost::function<void(int)> f2; // int 参数，无返回值
f2 = boost::bind(&Foo::methodInt, &foo, _1);
f2(53); // 调用 foo.methodInt(53);
```
如果没有boost::bind，那么boost::function就什么都不是，而有了bind()，“同一个类的不同对象可以delegate给不同的实现，从而实现不同的行为”（myan语），简直就无敌了。

## 对程序库的影响
程序库的设计不应该给使用者带来不必要的限制（耦合），而继承是仅次于最强的一种耦合（最强耦合的是友元）。如果一个程序库限制其使用者必须从某个class派生，那么我觉得这是一个糟糕的设计。不巧的是，目前有些程序库就是这么做的。
### 例1：线程库
**常规OO设计：**
写一个Thread base class，含有（纯）虚函数 Thread#run()，然后应用程序派生一个继承class，覆写run()。程序里的每一种线程对应一个Thread的派生类。例如Java的Thread可以这么用。

缺点：如果一个class的三个method需要在三个不同的线程中执行，就得写helper class(es)并玩一些OO把戏。

**基于closure的设计：**
令Thread是一个具体类，其构造函数接受Callable对象。应用程序只需提供一个Callable对象，创建一份Thread实体，调用Thread#start()即可。Java的Thread也可以这么用，传入一个Runnable对象。C#的Thread只支持这一种用法，构造函数的参数是delegate ThreadStart。boost::thread也只支持这种用法。
```cpp
// 一个基于 closure 的 Thread class 基本结构
class Thread 
{ 
 public: 
  typedef boost::function<void()> ThreadCallback; 
  Thread(ThreadCallback cb) : cb_(cb) 
  { } 
  void start() 
  { 
    /* some magic to call run() in new created thread */ 
  } 
 private: 
  void run() 
  { 
    cb_(); 
  } 
  ThreadCallback cb_; 
  // ... 
}; 
```
使用：
```cpp
class Foo
{
 public:
  void runInThread();
};

Foo foo;
Thread thread(boost::bind(&Foo::runInThread, &foo));
thread.start();
```
### 例2：网络库
以boost::function作为桥梁，NetServer class对其使用者没有任何类型上的限制，只对成员函数的参数和返回类型有限制。使用者EchoService也完全不知道NetServer的存在，只要在main()里把两者装配到一起，程序就跑起来了。

```cpp
// library
class Connection;
class NetServer : boost::noncopyable
{
 public:
  typedef boost::function<void (Connection*)> ConnectionCallback;
  typedef boost::function<void (Connection*, const void*, int len)> MessageCallback;
  NetServer(uint16_t port);
  ~NetServer();
  void registerConnectionCallback(const ConnectionCallback&);
  void registerMessageCallback(const MessageCallback&);
  void sendMessage(Connection*, const void* buf, int len);
 private:
  // ...
};


// user
class EchoService
{
 public:
  typedef boost::function<void(Connection*, const void*, int)> SendMessageCallback; // 符合NetServer::sendMessage的原型

  EchoService(const SendMessageCallback& sendMsgCb)
    : sendMessageCb_(sendMsgCb)
  { }

  void onMessage(Connection* conn, const void* buf, int size) // 符合NetServer::NetServer::MessageCallback的原型
  {
    printf("Received Msg from Connection %d: %.*s/n", conn->id(), size, (const char*)buf);
    sendMessageCb_(conn, buf, size); // echo back
  }

  void onConnection(Connection* conn) // 符合NetServer::NetServer::ConnectionCallback的原型
  {
    printf("Connection from %s:%d is %s/n", conn->ipAddr(), conn->port(), conn->connected() ? "UP" : "DOWN");
  }

 private:
  SendMessageCallback sendMessageCb_;
}; 

// 扮演上帝的角色，把各部件拼起来
int main()
{
  NetServer server(7);
  EchoService echo(bind(&NetServer::sendMessage, &server, _1, _2, _3));
  server.registerMessageCallback(bind(&EchoService::onMessage, &echo, _1, _2, _3));
  server.registerConnectionCallback(bind(&EchoService::onConnection, &echo, _1));
  server.run();
}
```

## 对面向对象程序设计的影响
一直以来，我对面向对象有一种厌恶感，叠床架屋，绕来绕去的，一拳拳打在棉花上，不解决实际问题。面向对象三要素是封装、继承和多态。我认为封装是根本的，继承和多态则是可有可无。用class来表示concept，这是根本的；至于继承和多态，其耦合性太强，往往不划算。

继承和多态不仅规定了函数的名称、参数、返回类型，还规定了类的继承关系。在现代的OO编程语言里，借助反射和attribute/annotation，已经大大放宽了限制。举例来说，JUnit 3.x 是用反射，找出派生类里的名字符合 void test*() 的函数来执行，这里就没继承什么事，只是对函数的名称有部分限制（继承是全面限制，一字不差）。至于JUnit 4.x 和 NUnit 2.x 则更进一步，以annoatation/attribute来标明test case，更没继承什么事了。

我的猜测是，当初提出面向对象的时候，closure还没有一个通用的实现，所以它没能算作基本的抽象工具之一。现在既然closure已经这么方便了，或许我们应该重新审视面向对象设计，至少不要那么滥用继承。

自从找到了boost::function+boost::bind这对神兵利器，不用再考虑类直接的继承关系，只需要基于对象的设计(object-based)，拳拳到肉，程序写起来顿时顺手了很多。

## 对面向对象设计模式的影响
既然虚函数能用closure代替，那么很多OO设计模式，尤其是行为模式，失去了存在的必要。另外，既然没有继承体系，那么创建型模式似乎也没啥用了。

最明显的是Strategy，不用累赘的Strategy基类和ConcreteStrategyA、ConcreteStrategyB等派生类，一个boost::function<>成员就解决问题。在《设计模式》这本书提到了23个模式，我认为iterator有用（或许再加个State），其他都在摆谱，拉虚架子，没啥用。或许它们解决了面向对象中的常见问题，不过要是我的程序里连面向对象（指继承和多态）都不用，那似乎也不用叨扰面向对象设计模式了。

或许closure-based programming将作为一种新的programming paradiam而流行起来。


## 依赖注入与单元测试
前面的EchoService可算是依赖注入的例子，EchoService需要一个什么东西来发送消息，它对这个“东西”的要求只是函数原型满足SendMessageCallback，而并不关系数据到底发到网络上还是发到控制台。在正常使用的时候，数据应该发给网络，而在做单元测试的时候，数据应该发给某个DataSink。

安照面向对象的思路，先写一个AbstractDataSink interface，包含sendMessage()这个虚函数，然后派生出两个classes：NetDataSink和MockDataSink，前面那个干活用，后面那个单元测试用。EchoService的构造函数应该以AbstractDataSink*为参数，这样就实现了所谓的接口与实现分离。

我认为这么做纯粹是脱了裤子放屁，直接传入一个SendMessageCallback对象就能解决问题。在单元测试的时候，可以boost::bind()到MockServer上，或某个全局函数上，完全不用继承和虚函数，也不会影响现有的设计。


## 什么时候使用继承？
如果是指OO中的public继承，即为了接口与实现分离，那么我只会在派生类的数目和功能完全确定的情况下使用。换句话说，不为将来的扩展考虑，这时候面向对象或许是一种不错的描述方法。一旦要考虑扩展，什么办法都没用，还不如把程序写简单点，将来好大改或重写。

如果是功能继承，那么我会考虑继承boost::noncopyable或boost::enable_shared_from_this，下一篇blog会讲到enable_shared_from_this在实现多线程安全的Signal/Slot时的妙用。

例如，IO-Multiplex在不同的操作系统下有不同的推荐实现，最通用的select()，POSIX的poll()，Linux的epoll()，FreeBSD的kqueue等等，数目固定，功能也完全确定，不用考虑扩展。那么设计一个NetLoop base class加若干具体classes就是不错的解决办法。


## 基于接口的设计
这个问题来自那个经典的讨论：不会飞的企鹅（Penguin）究竟应不应该继承自鸟（Bird），如果Bird定义了virtual function fly()的话。讨论的结果是，把具体的行为提出来，作为interface，比如Flyable（能飞的），Runnable（能跑的），然后让企鹅实现Runnable，麻雀实现Flyable和Runnable。（其实麻雀只能双脚跳，不能跑，这里不作深究。）

进一步的讨论表明，interface的粒度应足够小，或许包含一个method就够了，那么interface实际上退化成了给类型打的标签(tag)。在这种情况下，完全可以使用boost::function来代替，比如：
```cpp
// 企鹅能游泳，也能跑
class Penguin
{
 public:
  void run();
  void swim();
};

// 麻雀能飞，也能跑
class Sparrow
{
 public:
  void fly();
  void run();
};

// 以 closure 作为接口
typedef boost::function<void()> FlyCallback;
typedef boost::function<void()> RunCallback;
typedef boost::function<void()> SwimCallback;

// 一个既用到run，也用到fly的客户class
class Foo
{
 public:
  Foo(FlyCallback flyCb, RunCallback runCb) : flyCb_(flyCb), runCb_(runCb)
  { }

 private:
  FlyCallback flyCb_;
  RunCallback runCb_;
};
 
// 一个既用到run，也用到swim的客户class
class Bar
{
 public:
  Bar(SwimCallback swimCb, RunCallback runCb) : swimCb_(swimCb), runCb_(runCb)
  { }

 private:
  SwimCallback swimCb_;
  RunCallback runCb_;
};

int main()
{
  Sparrow s;
  Penguin p;
  // 装配起来，Foo要麻雀，Bar要企鹅。
  Foo foo(bind(&Sparrow::fly, &s), bind(&Sparrow::run, &s));
  Bar bar(bind(&Penguin::swim, &p), bind(&Penguin::run, &p));
}
```
## 实现Signal/Slot
boost::function + boost::bind 描述了一对一的回调，在项目中，我们借助boost::shared_ptr + boost::weak_ptr简洁地实现了多播(multi-cast)，即一对多的回调，并且考虑了对象的生命期管理与多线程安全；并且，自然地，对使用者的类型不作任何限制，篇幅略长，留作下一篇blog吧。（boost::signals也实现了Signal/Slot，但可惜不是线程安全的。）

 

最后，向伟大的C语言致敬！
