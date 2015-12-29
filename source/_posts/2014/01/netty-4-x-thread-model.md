title: Netty 4.x学习笔记 - 线程模型
date: 2014-01-14 23:25:00
categories: 技术分享
tags: [Netty, 线程模型]
---

### 1、前言

前面两篇学习笔记已经说完了[ByteBuf](http://hongweiyi.com/2014/01/netty-4-x-bytebuf/)和[Channel和Pipeline](http://hongweiyi.com/2014/01/netty-4-x-channel-pipeline/)，这篇开始讲讲前面欠的债——线程模型（EventLoop和EventExecutor）。

<!--more-->


### 2、Netty线程模型

将具体代码实现前，先来谈谈Netty的线程模型。正如许多博客所提到的，Netty采用了Reactor模式，但是许多博客也只是提到了而已，同时大家也不会忘记附上几张Doug Lee大神的图，但是并不会深入的解释。为了更好的学习和理解Netty的线程模型，我在这里稍微详细的说一下我对它的理解。

Reactor模式有多个变种，Netty基于Multiple Reactors模式（如下图）做了一定的修改，Mutilple Reactors模式有多个reactor：mainReactor和subReactor，其中mainReactor负责客户端的连接请求，并将请求转交给subReactor，后由subReactor负责相应通道的IO请求，非IO请求（具体逻辑处理）的任务则会直接写入队列，等待worker threads进行处理。

<center><div style="width: 80%;">![Multiple Reactors](/images/wpid-Multi-reactors3.png)</div></center>

Netty的线程模型基于Multiple Reactors模式，借用了mainReactor和subReactor的结构，但是从代码里看来，它并没有Thread Pool这个东东。Netty的subReactor与worker thread是同一个线程，采用IO多路复用机制，可以使一个subReactor监听并处理多个channel的IO请求，我给称之为：「Single Thread with many Channel」。我根据代码整理出下面这种Netty线程模型图：

<center><div style="width: 80%;">![Netty线程模型](/images/wpid-Netty-thread-model3.png)</div></center>

上图中的parentGroup和childGroup是Bootstrap构造方法中传入的两个对象，这两个group均是线程池，childGroup线程池会被各个subReactor充分利用，parentGroup线程池则只是在bind某个端口后，获得其中一个线程作为mainReactor。上图我将subReactor和worker thread合并成了一个个的loop，具体的请求操作均在loop中完成，下文会对loop有个稍微详细的解释。&nbsp;

>	以上均是Nio情况下。Oio采用的是Thread per Channel机制，即每个连接均创建一个线程负责该连接的所有事宜。
>	Doug Lee大神的Reactor介绍：[Scalable IO in Java](http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)

### 3、EventLoop和EventExecutor实现

EventLoop和EventExecutor实现共有4个主要逻辑接口，EventLoop、EventLoopGroup、EventExecutor、EventExecutorGroup，内部实现、继承的逻辑表示无法直视，有种擦边球的感觉。具体的类图如下：

<center><div style="width: 80%;">![EventLoop和EventExecutor类图](/images/wpid-EventLoopAndEventExecutor3.jpg)</div></center>

#### 3.1 EventLoopGroup:

主要方法是newChild，我理解为EventLoop的工厂类。`**EventLoopGroup.newChild`创建`**EventLoop`对象。OioEventLoopGroup除外，它没有实现newChild方法，调用父类的并创建ThreadPerChannelEventLoop对象。

#### 3.2 EventLoop:

主要方法是run()，是整个Netty执行过程的逻辑代码实现，后面细说。

#### 3.3 EventExecutorGroup:

线程池实现，主要成员是children数组，主要方法是next()，获得线程池中的一个线程，由子类调用。由于Oio采用的是Thread per Channel机制，所以没有实现前面两个。

#### 3.4 EventExecutor:

Task的执行类，主要成员是taskQueue以及真正的运行线程对象executor，主要方法是taskQueue操作方法execute、takeTask、addTask等，以及doStartThread方法，后面细说。

### 4、NioEventLoopGroup实现

这里以常用的NioEventLoopGroup为例。NioEventLoopGroup在Bootstrap初始化时作为参数传入构造方法，由于NioEventLoopGroup涉及的代码较多，就不大篇幅的贴代码了，只写流程性的文字或相应类和方法：

#### 4.1 mainReactor:

`1. Bootstrap.bind(port)`
`2. Bootstrap.initAndRegister()`
`2.1 Boostrap.init()`

> 初始化Channel，配置Channel参数，以及Pipeline。其中初始化Pipeline中，需要插入ServerBootstrapAcceptor对象用作acceptor接收客户端连接请求，acceptor也是一种ChannelInboundHandlerAdapter。

``` java
p.addLast(new ChannelInitializer<Channel>() {
  @Override
  public void initChannel(Channel ch) throws Exception {
    ch.pipeline().addLast(new ServerBootstrapAcceptor(currentChildHandler, currentChildOptions,
       currentChildAttrs));
  }
});
```

> 调用channel的unsafe对象注册selector，具体实现类为AbstractChannel$AbstractUnsafe.register。如下：

``` java
public final void register(final ChannelPromise promise) {
  if (eventLoop.inEventLoop()) {  // 是否在Channel的loop中
    register0(promise);
  } else {  // 不在
    try {
      eventLoop.execute(new Runnable() {  // EventLoop执行一个任务
        @Override
        public void run() {
          register0(promise);
        }
      });
    } catch (Throwable t) {
    // ...
    }
  }
}
```

> eventLoop.execute(runnable);是比较重要的一个方法。在没有启动真正线程时，它会启动线程并将待执行任务放入执行队列里面。启动真正线程(startThread())会判断是否该线程已经启动，如果已经启动则会直接跳过，达到线程复用的目的。启动的线程，主要调用方法是NioEventLoop的run()方法，run()方法在下面有详细介绍：

``` java
public void execute(Runnable task) {
  if (task == null) {
    throw new NullPointerException(&quot;task&quot;);
  }

  boolean inEventLoop = inEventLoop();
  if (inEventLoop) {
    addTask(task);
  } else {
    startThread();  // 启动线程
    addTask(task);  // 添加任务队列

    // ...

  }

  if (!addTaskWakesUp) {
    wakeup(inEventLoop);
  }
}
```

   2.2 group().register(channel)

> 将 channel 注册到下一个 EventLoop 中。

`3. 接收连接请求`

由NioEventLoop.run()接收到请求：

`3.1 AbstractNioMessageChannel$NioMessageUnsafe.read()`

`3.2 NioServerSocketChannel.doReadMessages()`

> 获得childEventLoopGroup中的EventLoop，并依据该loop创建新的SocketChannel对象。

`3.3 pipeline.fireChannelRead(readBuf.get(i));`

> readBuf.get(i)就是3.2中创建的SocketChannel对象。在2.2初始化Bootstrap的时候，已经将acceptor处理器插入pipeline中，所以理所当然，这个SocketChannel对象由acceptor处理器处理。

`3.4 ServerBootstrapAcceptor$ServerBootstrapAcceptor.channelRead();`

> 该方法流程与2.2、2.3类似，初始化子channel，并注册到相应的selector。注册的时候，也会调用eventLoop.execute用以执行注册任务，execute时，启动子线程。即启动了subReactor。

#### 4.2 subReactor:

subReactor的流程较为简单，主体完全依赖于loop，用以执行read、write还有自定义的NioTask操作，就不深入了，直接跳过解释loop过程。

**loop:**

loop是我自己提出来的组件，仅是代表subReactor的主要运行逻辑。例子可以参考NioEventLoop.run()。

loop会不断循环一个过程：select -&gt; processSelectedKeys(IO操作) -&gt; runAllTasks(非IO操作)，如下代码：

``` java
protected void run() {
  for (;;) {
    // ...
    try {
      if (hasTasks()) { // 如果队列中仍有任务
        selectNow();
      } else {
        select();
        // ...
      }

      // ...

      final long ioStartTime = System.nanoTime();  // 用以控制IO任务与非IO任务的运行时间比
      needsToSelectAgain = false;
      // IO任务
      if (selectedKeys != null) {
        processSelectedKeysOptimized(selectedKeys.flip());
      } else {
        processSelectedKeysPlain(selector.selectedKeys());
      }
      final long ioTime = System.nanoTime() - ioStartTime;

      final int ioRatio = this.ioRatio;
      // 非IO任务
      runAllTasks(ioTime * (100 - ioRatio) / ioRatio);

      if (isShuttingDown()) {
        closeAll();
        if (confirmShutdown()) {
          break;
        }
      }
    } catch (Throwable t) {
    // ...
    }
  }
}
```


就目前而言，基本上IO任务都会走processSelectedKeysOptimized方法，该方法即代表使用了优化的SelectedKeys。除非采用了比较特殊的JDK实现，基本都会走该方法。

> 1. selectedKeys在openSelector()方法中初始化，Netty通过反射修改了Selector的selectedKeys成员和publicSelectedKeys成员。替换成了自己的实现&mdash;&mdash;SelectedSelectionKeySet。
> 2. 从OpenJDK 6/7的SelectorImpl中可以看到，selectedKeys和publicSeletedKeys均采用了HashSet实现。HashSet采用HashMap实现，插入需要计算Hash并解决Hash冲突并挂链，而SelectedSelectionKeySet实现使用了双数组，每次插入尾部，扩展策略为double，调用flip()则返回当前数组并切换到另外一个数据。
> 3. ByteBuf中去掉了flip，在这里是否也可以呢？

processSelectedKeysOptimized主要流程如下：

``` java
final Object a = k.attachment();

if (a instanceof AbstractNioChannel) {
  processSelectedKey(k, (AbstractNioChannel) a);
} else {
  @SuppressWarnings(&quot;unchecked&quot;)
  NioTask<SelectableChannel> task = (NioTask<SelectableChannel>) a;
  processSelectedKey(k, task);
}
```

在获得attachment后，判断是Channel呢还是其他，其他则是NioTask。找遍代码并没有发现Netty有注册NioTask的行为，同时也没发现NioTask的实现类。只有在NioEventLoop.register方法中有注册NioTask至selector的行为，便判断该行为是由用户调用，可以针对某个Channel注册自己的NioTask。这里就只讲第一个processSelectdKey(k, (AbstractNioChannel) a)，但代码就不贴了。

和常规的NIO代码类似，processSelectdKey是判断SeletedKeys的readyOps，并做出相应的操作。操作均是unsafe做的。如read可以参考：AbstractNioByteChannel$NioByteUnsafe.read()。IO操作的流程大致都是：

* 获得数据
* 调用pipeline的方法，`fireChannel***`
* 插入任务队列

执行完所有IO操作后，开始执行非IO任务（runAllTasks）。Netty会控制IO和非IO任务的比例，ioTime * (100 - ioRatio) / ioRatio，默认ioRatio为50。runAllTasks乃是父类SingleThreadExecutor的方法。方法主体很简单，将任务从TaskQueue拎出来，直接调用任务的run方法即可。

>	代码调用的是task.run()，而不是task.start()。即是单线程执行所有任务

``` java
protected boolean runAllTasks(long timeoutNanos) {
  fetchFromDelayedQueue();
  Runnable task = pollTask();
  if (task == null) {
    return false;
  }

  // 控制时间
  final long deadline = ScheduledFutureTask.nanoTime() + timeoutNanos;
  long runTasks = 0;
  long lastExecutionTime;
  for (;;) {
    try {
      task.run();
    } catch (Throwable t) {
      logger.warn(&quot;A task raised an exception.&quot;, t);
    }

    runTasks ++;

    // Check timeout every 64 tasks because nanoTime() is relatively expensive.
    // XXX: Hard-coded value - will make it configurable if it is really a problem.
    if ((runTasks & 0x3F) == 0) {
        lastExecutionTime = ScheduledFutureTask.nanoTime();
      if (lastExecutionTime >= deadline) {
        break;
      }
    }

    task = pollTask();
    if (task == null) {
     lastExecutionTime = ScheduledFutureTask.nanoTime();
     break;
    }
  }

  this.lastExecutionTime = lastExecutionTime;
  return true;
}
```

### 5、总结

以上内容从设计和代码层面总结Netty线程模型的大致内容，中间有很多我的不成熟的思考与理解，请朋友轻拍与指正。

看源码过程中是比较折磨人的。首先得了解你学习东西的业务价值是哪里？即你学了这个之后能用在哪里，只是不考虑场景仅仅为了看代码而看代码比较难以深入理解其内涵；其次，看代码一定一定得从逻辑、结构层面看，从细节层面看只会越陷越深，有种一叶障目不见泰山的感觉；最后，最好是能够将代码逻辑、结构画出来，或者整理出思维导图啥的，可以用以理清思路。前面两篇文章思维道路较为清晰，线程模型的导图有一些但是比较混乱，就不贴出来了，用作自己参考，有兴趣的可以找我要噢。
