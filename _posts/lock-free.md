---
title: lock-free
toc: true
comments: true
date: 2017-07-13 19:47:51
tags: lock-free
categories: 并发编程
---

所谓`lock-free`，也即锁无关，通俗的解释如下

>  一个“锁无关”的程序能够确保执行它的所有线程中至少有一个能够继续往下执行。”

<!--more-->

![](/uploads/lock-free.png)

多线程并发编程中，如果多个线程之间存在共享内存，为了保证线程安全，每个线程存取共享内存的时候都要显式加锁。当有一个线程取得了锁，它就拿到了共享内存的存取权限，其他没有取到锁的线程需要原地阻塞，直到占有锁的线程主动释放锁，同时自己拿到相应的锁。

考虑一种情况，假设拿到锁的线程突然挂了，而且没有释放锁，这种情况下整个程序都会阻塞在这里。而`lock-free`正是可以避免这种情况，在`lock-free`下，即使有线程挂掉，也不影响整个程序的继续往下运行，也就是说，系统整体是一直在前进的。

上面说了，显式加锁保证线程安全的同时会出现非`lock-free`的情况，但是不加锁也会导致着中情况，例如下面的代码：

```c++
while (x == 0) {
	x = 1 - x;
}
```

如果两个线程同时执行进入while，然后x两次改变值，还是为0，这样的话两个线程都会一直阻塞在while，系统整体无法前进，依然不是`lock-free`。

关于`lock-free`的讨论，*Andrei Alexandrescu*曾在他的专栏**Generic<Programming>**写过一片文章，《[Lock-Free Data Structures](http://www.ddj.com/cpp/184401865)》，详细讲解了`lock-free`技术，以下是中文译文。



在Generic<Programming>沉默了一期之后(研究生的学业总是使人不得不投入百分之百的精力),这一期文章的可写内容突然多得令人似乎有点无所适从.例如,其中之一就是关于构造函数的讨论,特别是转发构造函数(forwarding constructor),(构造函数中的)异常处理,以及两段式(two-stage)对象构造.另一个可选主题是创建不完全类型的容器(例如list、vector或map)（顺便再回顾一下Yanslander技术[2]），这一任务可以借助于一些有趣的技巧来完成（当下的标准库容器并不能保证可存放不完全类型的对象）。

虽说以上两个候选主题都挺诱人，不过跟所谓的“锁无关（Lock-Free）”数据结构一比就只能靠边站了，后者是多线程编程的极重要技术之一。在今年的“Programming Language Design and Implementation”大会([http://www.cs.umd.edu/~pugh/pldi04/](http://www.cs.umd.edu/~pugh/pldi04/))上，Michael Maged展示了世界上第一个锁无关的(lock-free)内存分配器[7]，跟那些小心翼翼地设计的、基于锁(lock-based)的，同时也更为复杂的内存分配器相比，Michael的锁无关的内存分配器在诸多[测试](http://lib.csdn.net/base/softwaretest)下都表现得更为优越。

Michael的锁无关内存分配器代表着近来出现的许多锁无关数据结构和[算法](http://lib.csdn.net/base/datastructure)的最新进展。

****

### 什么是“锁无关(lock-free)”？

不久前这个问题正是我想要问的。作为一个真正的主流多线程程序员，我对基于锁的多线程算法并不感到陌生。在经典的基于锁的多线程程序设计中，只要你需要共享某些数据，你就应当将对它的访问串行化。修改（共享）数据的操作必须以原子操作的形式出现，这样才能保证没有其它线程能在中途插一脚来破坏相应数据的不变式（invariant）。即便像++count_(count_是整型变量)这样的简单操作也得加锁，因为增量操作实际上是分三步进行的：读、改、写（回），而这显然不是原子的。

简而言之，在基于锁的多线程编程中，你得确保任何针对共享数据的、且有可能导致竞争条件（race conditions）的操作都被做成了原子操作（通过对一个互斥体（mutex）进行加锁解锁）。从好的一面来说，只要互斥体是在锁状态，你就可以放心地进行任何操作，不用担心其它线程会闯进来搞坏你的共享数据。

然而，正是这种在互斥体的锁状态下可以为所欲为的事实同样也带来了问题。例如，你可以在锁期间读键盘或进行某些耗时较长的I/O操作，这便意味着其它想要占用你正占用着的互斥体的线程只能被晾在一旁等着。更糟的是，此时你可能会试图对另一个互斥体进行加锁，后者彼时或许已经被另一个线程占用了，而这个线程倒过来又试图去获取已然被你的线程所占用的互斥体，如此一来两个线程全都陷在那里动弹不得，死锁！

而在锁无关多线程编程的世界里，几乎任何操作都是无法原子地完成的。只有很小一集操作可以被原子地进行，这一限制使得锁无关编程的难度大大地增加了。（实际上，世界上肯定有不少锁无关编程专家，而我则不在此列。不过幸运的是，这篇文章中所提供的基本工具、内容以及热情可能会激发你成为一个这方面的专家。）锁无关编程带来的好处是在线程进展和线程交互方面，借助于锁无关编程，你能够对线程的进展和交互提供更好的保证。但话说回来，刚刚我们所谓的“很小一集可以被原子地进行的操作”又是指哪些操作呢？实际上，这个问题相当于，最少需要一集什么样的原语才能够允许我们实现出任何锁无关的算法（如果存在这么一个“最小集”的话）？

如果你认为这个问题的基础性使得它的解决者理当获得奖赏的话，你并不是唯一一个这么认为的。2003年，Maurice Herlihy因他在1991年发表的开创性论文“Wait-Free Synchronization”（ [http://www.podc.org/dijkstra/2003.html](http://www.podc.org/dijkstra/2003.html)）而获得了分布式编程的Edsger W. Dijkstra奖。在论文中，Herlihy证明了哪些原语对于构造锁无关数据结构来说是好的，哪些则是不好的。他的论述使得一些看似漂亮的硬件架构突然变得一钱不值，同时他的论文还指明了在将来的硬件中应当实现哪些同步原语。

例如，Herlihy的论文指出，像test-and-set、swap、fetch-and-add、甚至原子队列这样的看似强大的工具都不足以良好地同步两个以上的线程（这一结果很是令人惊讶，因为带有原子push和pop操作的队列看上去提供了一个相当强大的抽象）。从好的一面来说，Herlihy同样给出了一般性结论，他证明了一些简单的结构就足以实现出任何针对任意数目的线程的锁无关算法。

最简单、也是最普遍的一个通用原语就是CAS操作（Compare-And-Swap），这也是我一直使用的原语：

```c++
template <class T>
bool CAS(T* addr, T expected, T value) {
	if (*addr == expected) {
		*addr = value;
		return true;
	}

     return false;
} 
```

CAS原语负责比较某个内存地址处的内容与一个期望值，如果比较成功则将该内存地址处的内容替换为一个新值。这整个操作是原子的。许多现代处理器都实现了不同位长度的CAS或其等价原语（这里我用模板来表达它正是由于这个原因，我们可以假定一个具体的实现使用元编程来限制可能的T）。作为一个原则，CAS能够操作的位数越多，使用它来实现锁无关数据结构就越容易。如今的32位处理器实现了64位的CAS，例如Intel汇编指令中称它为CMPXCHG8（你得跟这些汇编助记符多亲近亲近）。

### 一点告诫

通常一篇C++文章中会伴随着C++代码片断和示例。理想情况下这些代码会是符合标准C++规范的，而且Generic<Programming>专栏的确尽量做到了这一点。

然而，当文章涉及的内容是多线程编程时，给出符合标准C++的示例代码就是不可能的了，因为标准C++中根本就没有线程的概念，而你无法编码不存在的东西。因此，这篇文章中的代码某种意义上只不过是“伪码”，而并非可移植的标准C++代码。例如内存屏障（memory barriers），对于本文中描述的算法来说，真正的实现代码要么是汇编写的，要么得在C++代码中到处插入所谓的“内存屏障”（“内存屏障”是一种处理器相关的机制，它能够强制内存读写的顺序性）。为了不失讨论重点，这里我并不打算对内存屏障多作介绍，有兴趣的读者可以参考Butenhof的著作[3]，或者在下面的注中提到的一篇关于它的简单介绍[6]。为了方便讨论，这里我假定我们的编译器和硬件都没有进行过分的优化（例如消除某些“冗余”的变量读取，这在单线程环境下的确是个有效的优化）。从技术上来讲，这即是说一个“顺序一致”的模型，其中读和写操作的执行顺序跟它们在源代码中的顺序是完全一样的[8]。

### 等待无关（Wait-Free）/锁无关（Lock-Free）与基于锁（Lock-Based）的比较

一个“等待无关”的程序可以在有限步之内结束，而不管其它线程的相对速度如何。

一个“锁无关”的程序能够确保执行它的所有线程中至少有一个能够继续往下执行。这便意味着有些线程可能会被任意地延迟，然而在每一步都至少有一个线程能够往下执行。因此这个系统作为一个整体总是在“前进”的，尽管有些线程的进度可能不如其它线程来得快。而基于锁的程序则无法提供上述的任何保证。一旦任何线程持有了某个互斥体并处于等待状态，那么其它任何想要获取同一互斥体的线程就只好站着干瞪眼；一般来说，基于锁的算法无法摆脱“死锁”或“活锁”的阴影，前者如两个线程互相等待另一个所占有的互斥体，后者如两个线程都试图去避开另一个线程的锁行为，就像两个在狭窄走廊里面撞了个照面的家伙，都试图去给对方让路，结果像跳交谊舞似的摆来摆去最终还是面对面走不过去。当然，对于我们人来说，遇到这种情况打个哈哈就行了，但处理器却不这么认为，它会不知疲倦地就这样干下去直到你强制重启。

等待无关和锁无关算法的定义意味着它们有更多的优点：

- 线程中止免疫：杀掉系统中的任何线程都不会导致其它线程被延迟。
- 信号免疫：C和C++标准禁止在信号或异步中断中调用某些系统例程（如malloc）。如果中断与某个被中断线程同时调用malloc的话，结果就会导致死锁。而锁无关例程则没有这一问题：线程可以自由地互相穿插执行。
- 优先级倒置免疫：所谓“优先级倒置”就是指一个低优先级线程持有了一个高优先级线程所需要的互斥体。这种微妙的冲突必须由OS内核来解决。而等待无关和锁无关算法则对此免疫。

### 一个锁无关的WRRM Map

写专栏文章的好处之一就是你可以定义自己的缩写，所以就让我们来定义WRRM（Write Rarely Read Many）这一缩写，一个WRRM Map是一个被读比被写的次数多得多的map。例如，对象工厂[1]，观察者模式的许多具体实例[5]，以及一个将币种跟汇率联系在一起的map（许多时候你只是对它进行读取，而只有很少的时候才会更新），当然还有许许多多其它的查找表。

WRRM Map可以通过std::map或“准标准”的unordered_map（[http://www.open-std.org/jtcl/sc22/wg21/docs/papers/2004/n1647.pdf](http://www.open-std.org/jtcl/sc22/wg21/docs/papers/2004/n1647.pdf)）来予以实现，不过正如我在Modern C++ Design中所说的，assoc_vector（一个已序的vector<pair>）也是一个不错的候选，因为它用更新速度换取了查找速度。总而言之，锁无关性质是与具体的数据结构选择无关（正交）的；我们姑且就称后者为Map<Key,Value>。同样，出于这篇文章的意图，当我说map时就是指那些提供了根据key来查找并能够更新key-value对的表，下文将不再重申这一点。

让我们来回顾一下一个基于锁的WRRMMap是怎样实现的：我们将一个Map对象与一个Mutex对象结合起来，像这样：

```c++
// WRRMMap的一个基于锁的实现
template <class K, class V>
class WRRMMap {
	Mutex mtx_;
	Map<K, V> map_;
public:
	V Lookup(const K& k) {
		Lock lock(mtx_);
		return map_[k];
	}

	void Update(const K& k, const V& v) {
		Lock lock(mtx_);
		map_[k] = v;
	}
};
```

为了避免所有权问题以及无效引用问题（这两个问题后面会给我们带来大麻烦），Lookup按值返回其结果。这一做法虽说稳妥但有代价。每次查找都会导致对Mutex加锁/解锁，而实际上一是平行的查找并不需要相互加锁，二是Update比Lookup发生的频率要小得多。那么，现在就让我们来实现一个更好的WRRMMap吧。

### 垃圾收集，你在何方？

对实现一个锁无关的WRRMMap的第一番尝试停在下面这几个问题上：

- 读取操作(Read)没有任何锁。
- 更新操作(Update)对整个map作一份拷贝，然后更新这份拷贝，最后再用这份拷贝来替换掉原来的旧map（通过CAS原语来进行）。如果最后一步的CAS操作没有成功的话，就循环尝试“拷贝/更新/CAS回去”这一步骤直到成功。
- 由于CAS原语所能操作的字节数是有限制的，所以WRRMMap存放指向实际Map的指针，而不是一个Map。

```c++
// WRRMMap的首个锁无关的实现
// 只有当你有GC的时候才行
template <class K, class V>
class WRRMMap {
	Map<K, V>* pMap_;
public:
	V Lookup (const K& k) {
		// 瞧，没有锁！
		 return (*pMap_) [k];
	}
	
  	void Update(const K& k, const V& v) {
		Map<K, V>* pNew = 0;
  		do {
			Map<K, V>* pOld = pMap_;
			delete pNew;
			pNew = new Map<K, V>(*pOld);
			(*pNew) [k] = v;
		} while (!CAS(&pMap_, pOld, pNew));
		// 别 delete pMap_;
	}
};
```

这行得通！在一个循环当中，Update()例程对整个map作了一份拷贝，并往该副本中加入了一个新项，最后再尝试交换新旧两个map的指针。这里，最后一步，一定要使用CAS原语而不是简单的赋值，这很关键，否则像下面这样的一个执行序列将会破坏该map：

- 线程A对该map作了一份副本
- 线程B对该map作了一份副本并更新了副本
- 线程A往其副本中加入新项
- 线程A用其改写后的副本替换原有map，这里，线程A的改写后的副本中没有线程B所作的任何改动。

而通过CAS，一切都能够优雅地完成，因为每个线程都相当于作了如下的陈述：“假设该map自从我上次观察它以来并没有任何更动，那么就进行CAS（写回改动后的新map）。否则一切从头来过。”

根据定义，这就使得Update成了一个锁无关的算法，但它并不是等待无关的。例如，如果若干线程同时调用Update，那么它们每一个都可能会循环尝试不确定的次数，然而总会有某个线程能够确保成功更新该map，因而整体上的进度总是在继续的。另外，幸运的是，Lookup是等待无关的。

在一个拥有垃圾收集的环境中，我们的工作就算完成了，而这篇文章也将在一片欢欣鼓舞中结束。然而，事实是我们并没有垃圾收集，因而只得面对麻烦了。麻烦就是，我们不能简单地就将旧的pMap_扔掉（意即delete——译注），如果正当你试图delete它的时候一帮其它线程冲上来想要访问它里面的数据（Lookup）该怎么办呢？你是知道的，垃圾收集器可以访问所有线程的数据及私有栈，所以它对于“哪些pMap_所指的map不再被使用了”这个问题有更清晰的认识，而且它也能够优雅地将这些不再被使用的map清理掉。而如果没有垃圾收集器的帮助，事情可就没那么简单了。实际上，是难得多！而且看起来确定性的内存归还在锁无关数据结构方面是个基础问题。

### 写锁定(Write-Locked)的WRRM Map

为了弄清我们遇到的问题有多棘手，让我们先来试试用经典的引用计数技术来实现WRRMMap，看看这有何不可。那么，考虑将一个引用计数器跟map的指针绑定在一起，并让WRRMMap保存一个指向如此构造出的数据结构的指针。

```c++
template <class K, class V>
class WRRMMap {
	typedef std::pair<Map<K, V>*, unsigned> Data;
 	Data* pData_;
	...
};
```

嗯，看上去不赖。现在，Lookup会先递增pData_->second，然后在map中进行查找，最后再递减pData_->second。当pData_->second计数器的值下降到0的时候，pData_->first就可以被delete掉了，接着pData_自己也就可以被delete掉了。看起来简单稳妥是不是，只不过…只不过它是幼稚的。试想下面这种情况，正当某个线程注意到引用计数器的值下降到0并着手delete pData_时，另一个线程或另一堆线程插进来拽住了垂死的pData_并准备从它进行读取！不管你怎么聪明地安排你的策略，你都逃不开一个基本的问题：即要想读取数据的指针，你得先递增引用计数，但引用计数又必须得是你要读取的数据的一部分，因此倘不先读取那个指针你就无法访问到该引用计数。这就好比一个将开关安放在其顶端的带电铁丝网：要想安全的爬上去你就得先将开关关掉，但要想将开关关掉你就得先安全地爬上去啊！

那么，就让我们来看看有没有其它方法能够正确地delete旧的map。一个方案是等待然后delete。考虑到随着处理器时间（毫秒计）的推移，旧的pMap_对象会被越来越少的线程所访问（这是因为新的访问是对新的map进行的），于是一旦那些在CAS操作之前开始的Lookup操作结束了，对应的（旧）pMap_也就可以去见阎王了。因此，我们的一个解决方案就是将旧的pMap_推入一个队列，并由一个专门的清理线程，每隔，比如说200毫秒，醒来一次，delete掉队列中待得最久的一个pMap_。

但这并非一个理论上安全的方案（尽管从实际上来说它可能绝大多数情况下都不会出什么意外）。比如说，由于某种原因一个进行Lookup的线程被延迟了，那么清理线程就会在该线程的眼皮子底下delete掉它正在访问的map。这可以通过总是给清理线程赋予一个比任何线程低的优先级来予以避免，然而总的来说，这个方案骨子里的劣根性是难以清除的。如果你也同意对于这样一个技术我们没法为其作任何正儿八经的辩护的话，我们就继续往下。

其它的方案[4]则依赖于一个扩展的DCAS原子操作，该操作能够compare-and-swap内存中的两个不必连续的字：

```c++
template <class T1, class T2>
bool  DCAS(T1* p1, T2* p2,
           T1 e1, T2 e2,
           T1 v1, T2 v2) {
	if (*p1 == e1 && *p2 == e2) {
		*p1 = v1; *p2 = v2;
		return true;
	}

	return false;
} 
```

这里所说的两个字当然是指map的指针以及引用计数器。DCAS已经由摩托罗拉68040处理器（非常低效地）实现了，然而其它处理器并没有实现它。因此，基于DCAS的解决方案主要是被当成理想方案来考虑的。

那么，在确定性析构的大背景下，我们对于该问题的尝试首先是求助于不像DCAS那么要求苛刻的CAS2操作。再次说明，许多32位的机器都实现了64位的CAS操作，被称为CAS2（CAS2仅操作连续的字，所以它的能力显然不及DCAS强大）。作为开始，让我们将引用计数跟它所“保卫”的map指针紧挨着存放在内存中：

```c++
template <class K, class V>
class WRRMMap {
	typedef std::pair<Map<K, V>*, unsigned> Data;
	Data data_;
	...
};
```

（注意，这一次，引用计数与它所保护的指针紧挨在一起了，这就消除了我们前面提到的“带电铁丝网-开关”的尴尬问题。待会你就会看到这样做的代价。）

那么，让我们修改Lookup从而使其能够在访问map之前先递增引用计数，并在访问结束之后将其递减回去。在下面的代码片断中，为了简洁起见，我并没有考虑异常安全问题（这个问题可以用标准技术来解决）。

```c++
V Lookup(const K& k) {
	Data old;
	Data fresh;
	do {
		old = data_;
		fresh = old;
		++fresh.second;
	} while (CAS(&data_, old, fresh));
	
	V temp = (*fresh.first)[k];
	
  	do {
		old = data_;
		fresh = old;
		--fresh.second;
	} while (CAS(&data_, old, fresh));

	return temp;
}
```

最后，Update负责将该map替换为一个新的——仅当其引用计数为1的时候。

```c++
void Update(const K& k, const V& v) {
	Data old;
	Data fresh;
  
	old.second = 1;
	fresh.first = 0;
	fresh.second = 1;

	Map<K, V>* last = 0;

  	do {
    	old.first = data_.first;
		if (last != old.first) {
			delete fresh.first;
			fresh.first = new Map<K, V>(old.first);
			fresh.first->insert(make_pair(k, v));
			last = old.first;
		}
	} while (!CAS(&data_, old, fresh));
	
  	delete old.first; // whew
}
```

Update是这样工作的：首先它定义old和fresh两个变量（我们现在应该非常熟悉这两个变量了）。只不过这次old.second并非由data_.second赋值而来，而是始终为1。这意味着，Update会一直循环直到它等到data_.first指针的引用计数变成1（此时没有任何线程在读），并将其替换为另一个引用计数为1的新map的指针。说白了这相当于如下的陈述：“我将会把旧的map替换成一个更新过的，而且我会留神其它对于该map的更新，不过，仅当现有map的引用计数为1的时候我才会进行替换。”变量last及其相关代码只不过是个优化技巧：当旧map并没有被改动，而只是其引用技术被改动的时候，last可以帮助避免我们重建同一个map。

挺漂亮的手法，对不对？只可惜事实并非如此。Update现在实际上是基于锁的：它得等待所有Lookup结束之后才能去更新该map。而锁无关数据结构的好处就在于一切都能够如行云流水般进行。特别地，在这种实现下，Update很容易就可以被饿死：你只需足够频繁地进行Lookup就行了，让它的引用计数永不降为1。总而言之，到目前为止我们得到的并非一个WRRM Map，而是一个WRRMBNTM（Write-Rarely-Read-Many-But-Not-Too-Many） Map。

### 结论

锁无关数据结构是大有前途的技术。它在线程中止、优先级倒置以及信号安全等方面都有着良好的表现。它们永远不会死锁或活锁。在测试当中，最近的锁无关数据结构远远超越了它们的基于锁的版本[9]。然而，锁无关编程的技巧性要求较高，特别是当涉及到内存释放的时候。一个垃圾收集环境对此是大有裨益的，因为它能够暂停并检视所有线程。不过，如果你想要确定性析构的话，你就需要来自硬件或内存分配器的特殊支持了。在下期的Generic<Programming>专栏中，我们将会考察优化WRRMMap的途径，使其在确定性析构的基础上实现锁无关。

如果本期的垃圾收集map和WRRMBNTM Map并没能满足你的胃口的话，下面是个省钱小诀窍：Don't Go watch the movie *Alien vs. Predator*, unless you like "so bad it's funny" movies.（译注：one of the crap jokes I do not get）。



**Acknowlegments**

David B. Held, Michael Maged, Larry Evans, and Scott Meyers provided very helpful feedback. Also, Michael graciously agreed to coauthor the next "Generic<Programming>" installment, which will greatly improve on our **WRRMap** implementation.

**References**

[1] Alexandrescu, Andrei. *Modern C++ Design*, Addison-Wesley Longman, 2001.

[2] Alexandrescu, Andrei. "Generic<Programming>:yasli::vector Is On the Move," *C/C++ Users Journal*, June 2004.

[3] Butenhof, D.R. *Programming with POSIX Threads*, Addison-Wesley, 1997.

[4] Detlefs, David L., Paul A. Martin, Mark Moir, and Guy L. Steele, Jr. "Lock-free Reference Counting," *Proceedings of the Twentieth Annual ACM Symposium on Principles of Distributed Computing*, pages 190-199, ACM Press, 2001. ISBN 1-58113-383-9.

[5] Gamma, Erich, Richard Helm, Ralph E. Johnson, and John Vlissides. *Design Patterns: Elements of Resusable Object-Oriented Software*, Addison-Wesley, 1995.

[6] Meyers, Scott and Andrei Alexandrescu. "The Perils of Double-Checked Locking." *Dr. Dobb's Journal*, July 2004.

[7] Maged, Michael M. "Scalable Lock-free Dynamic Memory Allocation," *Proceedings of the ACM SIGPLAN 2004 Conference on Programming Language Design and Implementation*, pages 35-46. ACM Press, 2004. ISBN 1-58113-807-5.

[8] Robison, Arch. "Memory Consistency & .NET," *Dr. Dobb's Journal*, April 2003.

[9] Maged, Michael M. "CAS-Based Lock-Free Algorithm for Shared Deques," *The Ninth Euro-Par Conference on Parallel Processing*, LNCS volume 2790, pages 651-660, August 2003.