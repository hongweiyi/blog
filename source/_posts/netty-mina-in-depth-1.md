title: netty-mina深入学习与对比（一）
date: 2014-05-17 21:42:00
categories: 技术分享
tags: [Java, Mina, Netty, 线程模型]

---

这博文的系列主要是为了更好的了解一个完整的nio框架的编程细节以及演进过程，我选了同父（Trustin Lee）的两个框架netty与mina做对比。版本涉及了netty3.x、netty4.x、mina1.x、mina2.x、mina3.x。这里并没有写netty5.x的细节，看了[netty5的修改文档](http://netty.io/wiki/new-and-noteworthy-in-5.x.html)，似乎有一些比较有意思的改动，准备单独写一篇netty4.x与netty5.x的不同。

<!--more-->

netty从twitter发布的这篇[Netty 4 at Twitter: Reduced GC Overhead](https://blog.twitter.com/2013/netty-4-at-twitter-reduced-gc-overhead)文章让国内Java界为之一振，也小火了一把，同时netty的社区发展也不错，版本迭代非常快，半年不关注大、小版本就发了好几轮了。但是mina就有点淡了，github上面它最后大多数代码最后的修改日期均在2013年，不过我从个人情感上还是挺喜欢mina3的代码，没有太多的用不上的功能（支持各种协议啥的），跑自带的benchmark性能也比netty4好一些。但是如果是生产用的话，就偏向netty多一些了，毕竟社区活跃，版本迭代也快。

### 1. mina、netty的线程模型

mina与netty都是Trustin Lee的作品，所以在很多方面都十分相似，他们线程模型也是基本一致，采用了Reactors in threads模型，即Main Reactor + Sub Reactors的模式。由main reactor处理连接相关的任务：accept、connect等，当连接处理完毕并建立一个socket连接（称之为session）后，给每个session分配一个sub reactor，之后该session的所有IO、业务逻辑处理均交给了该sub reactor。每个reactor均是一个线程，sub reactor中只靠内核调度，没有任何通信且互不打扰。

在前面的博文：[Netty 4.x学习笔记 – 线程模型](http://hongweiyi.com/2014/01/netty-4-x-thread-model/)，对netty的线程模型有一定的介绍。现在来讲讲我对线程模型演进的一些理解：

* **`Thread per Connection`**: 在没有nio之前，这是传统的java网络编程方案所采用的线程模型。即有一个主循环，socket.accept阻塞等待，当建立连接后，创建新的线程/从线程池中取一个，把该socket连接交由新线程全权处理。这种方案优缺点都很明显，优点即实现简单，缺点则是方案的伸缩性受到线程数的限制。
* **`Reactor in Single Thread`**: 有了nio后，可以采用IO多路复用机制了。我们抽取出一个单线程版的reactor模型，时序图见下文，该方案只有一个线程，所有的socket连接均注册在了该reactor上，由一个线程全权负责所有的任务。它实现简单，且不受线程数的限制。这种方案受限于使用场景，仅适合于IO密集的应用，不太适合CPU密集的应用，且适合于CPU资源紧张的应用上。

<center><div style="width: 30%;">![Reactor Single Thread](/images/reactor-single-thread.png)</div></center>

* **`Reactor + Thread Pool`**: 方案2由于受限于使用场景，但为了可以更充分的使用CPU资源，抽取出一个逻辑处理线程池。reactor仅负责IO任务，线程池负责所有其它逻辑的处理。虽然该方案可以充分利用CPU资源，但是这个方案多了进出thread pool的两次上下文切换。

<center><div style="width: 50%;">![Reactor + Thread Pool](/images/reactor-thread-pool.png)</div></center>

* **`Reactors in threads`**: 基于方案3缺点的考虑，将reactor分成两个部分。main reactor负责连接任务（accept、connect等），sub reactor负责IO、逻辑任务，即mina与netty的线程模型。该方案适应性十分强，可以调整sub reactor的数量适应CPU资源紧张的应用；同时CPU密集型任务时，又可以在业务处理逻辑中将任务交由线程池处理，如方案5。该方案有一个不太明显的缺点，即session没有分优先级，所有session平等对待均分到所有的线程中，这样可能会导致优先级低耗资源的session堵塞高优先级的session，但似乎netty与mina并没有针对这个做优化。

<center><div style="width: 60%;">![Reactors in threads](/images/reactors-in-threads.png)</div></center>

* **`Reactors in threads + Threads pool`**: 这也是我所在公司应用框架采用的模型，可以更为灵活的适应所有的应用场景：调整reactor数量、调整thread pool大小等。

<center><div style="width: 70%;">![Reactors in threads + Thread pool](/images/reactors-in-threads-thread-pool.png)</div></center>

> 以上图片及总结参考：《Linux多线程服务端编程》

### 2. mina、netty的任务调度粒度

mina、netty在线程模型上并没有太大的差异性，主要的差异还是在任务调度的粒度的不同。任务从逻辑上我给它分为成三种类型：连接相关的任务（bind、connect等）、写任务（write、flush）、调度任务（延迟、定时等），读任务则由selector加循环时间控制了。mina、netty任务调度的趋势是逐渐变小，从session级别的调度 -&gt; 类型级别任务的调度 -&gt; 任务的调度。

#### 2.1 代码

* `mina-1.1.7`: SocketIoProcessor$Worker.run
* `mina-2.0.4`: AbstractPollingIoProcessor$Processor.run
* `mina-3.0.0.M3-SNAPSHOT`: AbstractNioSession.processWrite
* `netty-3.5.8.Final`: AbstractNioSelector.run
* `netty-4.0.6.Final`: NioEventLoop.run

#### 2.2 分析

mina1、2的任务调度粒度为session。mina会将有IO任务的的session写入队列中，当循环执行任务时，则会轮询所有的session，并依次把session中的所有任务取出来运行。这样粗粒度的调度是不公平调度，会导致某些请求的延迟很高。

mina3的模型改动比较大，代码相对就比较难看了，我仅是随便扫了一下，它仅提炼出了writeQueue。

而netty3的调度粒度则是按照IO操作，分成了registerTaskQueue、writeTaskQueue、eventQueue三个队列，当有IO任务时，依次processRegisterTaskQueue、processEventQueue、processWriteTaskQueue、processSelectedKeys(selector.selectedKeys)。

netty4可能觉得netty3的粒度还是比较粗，将队列细分成了taskQueue和delayedTaskQueue，所有的任务均放在taskQueue中，delayedTaskQueue则是定时调度任务，且netty4可以灵活配置task与selectedKey处理的时间比例。

> BTW: netty3.6.0之后，所有的队列均合并成了一个taskQueue

有意思的是，netty4会优先处理selectedKeys，然后再处理任务，netty3则相反。mina1、2则是先处理新建的session，再处理selectedKeys，再处理任务。

> 难道selectedKeys处理顺序有讲究么？
