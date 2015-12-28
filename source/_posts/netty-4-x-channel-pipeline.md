title: Netty 4.x学习笔记 - Channel和Pipeline
date: 2014-01-07 17:51:00
categories: 技术分享
tags: [Netty]
---

### 1、前言

Channel概念与java.nio.channel概念一致，用以连接IO设备（socket、文件等）的纽带。Netty 4.x之后的Channel变化较大，官方的唬人的说法是无法通过简单的关键字替换进行迁移。用得较多应该是：ChannelHandler接口重新设计，换了个较为清晰的名字；write不会主动flush。由于笔者3.x、4.x都没用过，所以也无法深入理解版本的变化了。

<!--more-->

> 关于channel 4.x的新变化可以参考这里：new and noteworthy: Channel API changes

### 2、Channel总览

<center><div style="width: 80%;">![Netty Channel整体结构思维导图](/images/wpid-Channel.png)</div></center>

Channel的IO类型主要有两种：非阻塞IO（NIO）以及阻塞IO（OIO）；数据传输类型有两种：按事件消息传递（Message）以及按字节传递（Byte）；适用方类型也有两种：服务器（ServerSocket）以及客户端（Socket）。还有一些根据传输协议而制定的的Channel，如：UDT、SCTP等。

Netty按照类型逐层设计相应的类。最底层的为抽象类AbstractChannel，再以此根据IO类型、数据传输类型、适用方类型实现。类图可以一目了然，如下图所示：

<center><div style="width: 80%;">![Netty Channel类图](/images/wpid-nio-oio.jpg)</div></center>

### 3、ChannelPipeline实现分析

从AbstractChannel分析，它提供了一些IO操作方法，read、write等，Channel仅仅做了一个封装，方法中将参数直接传递给了Channel的Pipeline成员的相应方法。

Pipeline则是Channel里面非常重要的概念。从数据结构的角度，它是一个双向链表，每个节点均是DefaultChannelHandlerContext对象；从逻辑的角度，它则是netty的逻辑处理链，每个节点均包含一个逻辑处理器（ChannelHandler），用以实现网络通信的编/解码、处理等功能。

Pipeline的链表上有两种handler，Inbound Handler和Outbound handler。从Netty内部IO线程接读到IO数据，依次经过N个Handler到达最内部的逻辑处理单元，这种称之为Inbound Handler；从Channel发出IO请求，依次经过M个Handler到达Netty内部IO线程，这种称之为Outbound Handler。内部代码实现流程则是：Head -> Tail (Inbound)，Tail -> Head (Outbound)。下图截取自ChannelPipeline的注释中，简单明了：

<center><div style="width: 60%;">![Netty Pipeline](/images/wpid-Netty-ChannelPipeline-.png)</div></center>

### 4、逻辑处理器

ChannelPipeline仅仅只是逻辑处理的流程，真正逻辑处理器则是ChannelHandlerInvoker。在获得链表节点后，节点会调用自己的invoker成员执行(invoke)逻辑。

``` java
public ChannelFuture writeAndFlush(Object msg, ChannelPromise promise) {
    DefaultChannelHandlerContext next = findContextOutbound();
    next.invoker.invokeWriteAndFlush(next, msg, promise);
    return promise;
}
```

在DefaultChannelHandlerInvoker中只有一个成员(executor)，执行逻辑的过程中，Invoker会先判断当前运行线程是否是executor，如果是则直接运行相应方法，不是则启动子线程运行相应方法。

``` java
private void invokeWrite(ChannelHandlerContext ctx, Object msg, boolean flush, ChannelPromise promise) {

    if (executor.inEventLoop()) { // 判断是否是当前线程
        invokeWriteNow(ctx, msg, promise);
        if (flush) {
            invokeFlushNow(ctx);
        }
    } else {
        AbstractChannel channel = (AbstractChannel) ctx.channel();
        int size = channel.estimatorHandle().size(msg);
        if (size > 0) {
            ChannelOutboundBuffer buffer = channel.unsafe().outboundBuffer();
            // Check for null as it may be set to null if the channel is closed already
            if (buffer != null) {
                buffer.incrementPendingOutboundBytes(size);
            }
        }
        // 创建一个新的WriteTask
        // executor.execute(task);
        safeExecuteOutbound(WriteTask.newInstance(ctx, msg, size, flush, promise), promise, msg);
    }

}
```

executor继承自[EventExecutor](http://netty.io/4.0/api/io/netty/util/concurrent/EventExecutor.html)，同时，该对象实现类一般而言也是实现[EventLoop](http://netty.io/4.0/api/io/netty/channel/EventLoop.html)接口。EventLoop的实现体现了Netty 4.x的IO线程模型，非常重要，后面再详细解析。

#### 5、总结

至此，上面简单总结了Channel以及Pipeline的处理流程。`Channel.write -> ChannelPipeline.write -> ChannelHandlerContext.write -> ChannelHandlerInvoker.write -> ChannelHandler.write`。在这个过程中，我也是捡简单的、流程性的代码总结，像EventLoop、EventExecutor这种核心部分并没有深入总结，压后再详细解说。
