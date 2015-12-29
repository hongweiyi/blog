title: netty-mina深入学习与对比（二）
date: 2014-05-17 22:03:00
categories: 技术分享
tags: [Java, Mina, Netty]

---

上文讲了对netty-mina的线程模型以及任务调度粒度的理解，这篇则主要是讲nio编程中的注意事项，netty-mina的对这些注意事项的实现方式的差异，以及业务层会如何处理这些注意事项。

<!--more-->

### 1. 数据是如何write出去的
java nio如果是non-blocking的话，在每次write(bytes[N])的时候，并不会将N字节全部write出去，每次write仅一部分（具体大小和`tcp_write_buffer`有关）。那么，mina和netty是怎么处理这种情况的呢？

#### 1.1 代码

* `mina-1.1.7`: SocketIoProcessor.doFlush
* `mina-2.0.4`: AbstractPollingIoProcessor.flushNow
* `mina-3.0.0.M3-SNAPSHOT`: AbstractNioSession.processWrite
* `netty-3.5.8.Final`: AbstractNioWorker.write0
* `netty-4.0.6.Final`: AbstractNioByteChannel.doWrite

#### 1.2 分析

mina1、2，netty3的方式基本一致。 在发送端每个session均有一个writeBufferQueue，有这样一个队列，可以保证写入与写出均有序。在真正write时，大致逻辑均是一一将队列中的writeBuffer取出，写入socket。但有一些不同的是，mina1是每次peek一次，当该buffer全部写出之后再poll（mina3也是这种机制）；而mina2、netty3则是直接poll第一个，将其存为currentWriteRequest，直到currentWriteRequest全部写出之后，才会再poll下一个。这样的做法是为了省几次peek的时间么？

同时mina、netty在write时，有一种spin write的机制，即循环write多次。mina1的spin write count为256，写死在代码里了，表示256有点大；mina2这个机制废除但代码保留；netty3则可以配置，默认为16。netty在这里略胜一筹！

netty4与netty3的机制差不多，但是netty4为这个事情特意写了一个ChannelOutboundBuffer类，输出队列写在了该类的flushed:Object[]成员中，但表示ChannelOutboundBuffer这个类的代码有点长，就暂不深究了。

### 2. 数据是如何read进来的
如第三段内容，每次write只是输出了一部分数据，read同理，也有可能只会读入部分数据，这样就是导致读入的数据是残缺的。而mina和netty默认不会理会这种由于nio导致的数据分片，需要由业务层自己额外做配置或者处理。

#### 2.1 代码

* `nfs-rpc`: ProtocolUtils.decode
* `mina-1.1.7`: SocketIoProcessor.read, CumulativeProtocolDecoder.decode
* `mina-2.0.4`: AbstractPollingIoProcessor.read, CumulativeProtocolDecoder.decode
* `mina-3.0.0.M3-SNAPSHOT`: NioSelectorLoop.readBuffer
* `netty-3.5.8.Final`: NioWorker.read, FrameDecoder
* `netty-4.0.6.Fianl`: AbstractNioByteChannel$NioByteUnsafe.read

#### 2.2 业务层处理

nfs-rpc在协议反序列化的过程中，就会考虑这个的问题，依次读入每个字节，当发现当前字节或者剩余字节数不够时，会将buf的readerIndex设置为初始状态。具体的实现，有兴趣的同学可以学习`nfs-rpc：ProtocolUtils.decode`

nfs-rpc在decode时，出现错误就会将buf的readerIndex设为0，把readerIndex设置为0就必须要有个前提假设：每次decode时buf是同一个，即该buf是复用的。那么，具体情况是怎样呢？

#### 2.3 框架层处理

我看读mina与netty这块的代码，发现主要演进与不同的点在两个地方：读buffer的创建与数据分片的处理方式。

**mina:**

mina1、2的读buffer创建方式比较土，在每次read之前，会重新allocate一个新的buf对象，该buf对象的大小是根据读入数据大小动态调整。当本次读入数据等于该buf大小，下一次allocate的buf对象大小会翻倍；当本次读入数据不足该buf大小的二分之一，下一次allocate的buf对象同样会缩小至一半。需要注意的是，\*2与/2的代码都可以用位运算，但是mina1竟没用位运算，有意思。

mina1、2处理数据分片可以继承CumulativeProtocolDecoder，该decoder会在session中存入(BUFFER, cumulativeBuffer)。decode过程为：1）先将message追加至cumulativeBuffer；2）调用具体的decode逻辑；3）判断cumulativeBuffer.hasRemaining()，为true则压缩cumulativeBuffer，为false则直接删除(BUFFER, cumulativeBuffer)。实现业务的decode逻辑可以参考nfs-rpc中MinaProtocolDecoder的代码。

mina3在处理读buffer的创建与数据分片比较巧妙，它所有的读buffer共用一个buffer对象（默认64kb），每次均会将读入的数据追加至该buffer中，这样即省去了buffer的创建与销毁事件，也省去了cumulativeDecoder的处理逻辑，让代码很清爽啊！

**netty:**

netty3在读buffer创建部分的代码还是挺有意思的，首先，它创建了一个SocketReceiveBufferAllocator的allocate对象，名字为recvBufferPool，但是里面代码完全和pool扯不上关系；其次，它每次创建buffer也会动态修改初始大小的机制，它设计了232个大小档位，最大值为Integer.MAX_VALUE，没有具体考究，这种实现方式似乎比每次大小翻倍优雅一点，具体代码可以参考：`AdaptiveReceiveBufferSizePredictor`。

对应mina的CumulativeProtocolDecoder类，在netty中则是FrameDecoder和ReplayingDecoder，没深入只是大致扫了下代码，原理基本一致。BTW，ReplayingDecoder似乎挺强大的，有兴趣的可以看看这两篇：

> [High speed custom codecs with ReplayingDecoder](http://biasedbit.com/netty-tutorial-replaying-decoder)
> [An enhanced version of ReplayingDecoder for Netty](http://biasedbit.com/an-enhanced-version-of-replayingdecoder-for-netty/)

netty4在读buffer创建部分机制与netty3大同小异，不过由于netty有了ByteBufAllocator的概念，要想每次不重新创建销毁buffer的话，可以采用PooledByteBufAllocator。

在处理分片上，netty4抽象出了Message这样的概念，我的理解就是，一个Message就是业务可读的数据，转换Message的抽象类：ByteToMessageDecoder，当然也有netty3中的ReplayingDecoder，继承自ByteToMessageDecoder，具体可以研究代码。

### 3. ByteBuffer设计的差异

#### 3.1 自建buffer的原因

**mina:**

需要说明的是，只有mina1、2才有自己的buffer类，mina3内部只用nio的原生ByteBuffer类（提供了一个组合buffer的代理类-IoBuffer）。mina1、2自建buffer的原因如下：

* It doesn’t provide useful getters and putters such as fill,get/putString, and get/putAsciiInt()enough.
* It is difficult to write variable-length data due to its fixed capacity

第一条比较好理解，即提供了更为方便的方法用以操作buffer。第二条则是觉得nio的ByteBuffer是定长的，无法自动扩容或者缩容，所以提供了自动扩/缩容的方法：IoBuffer.setAutoExpand, IoBuffer.setAutoShrink。但是扩/缩容的实现，也是基于nio的ByteBuffer，重新ByteBuffer.allocate(capacity)，再把原有的数据拷贝过去。

**netty:**

在我前面的博文（[Netty 4.x学习笔记 – ByteBuf](http://hongweiyi.com/2014/01/netty-4-x-bytebuf/)）我已经提到这些原因：

* 需要的话，可以自定义buffer类型
* 通过组合buffer类型，可实现透明的zero-copy
* 提供动态的buffer类型，如StringBuffer一样(扩容方式也是每次double)，容量是按需扩展
* 无需调用flip()方法
* 常常「often」比ByteBuffer快

以上理由来自netty3的API文档：[Package org.jboss.netty.buffer](http://docs.jboss.org/netty/3.2/api/org/jboss/netty/buffer/package-summary.html)，netty4没见到官方的说法，但是我觉得还得加上一个更为重要也是最为重要的理由，就是可以实现buffer池化管理。

#### 3.2 实现的差异

**mina:**

mina的实现较为基础，仅仅只是在ByteBuffer上的一些简单封装。

**netty:**

netty3与netty4的实现大致相同（ChannlBuffer -&gt; ByteBuf），具体可以参见：[Netty 4.x学习笔记 – ByteBuf](http://hongweiyi.com/2014/01/netty-4-x-bytebuf/)，netty4实现了PooledByteBufAllocator，传闻是可以大大减少GC的压力，但是官方不保证没有内存泄露，我自己压测中也出现了内存泄露的警告，建议生产中谨慎使用该功能。

> netty5.x有一个更为高级的buffer泄露跟踪机制，PooledByteBufAllocator也已经默认开启，有机会可以尝试使用一下。
