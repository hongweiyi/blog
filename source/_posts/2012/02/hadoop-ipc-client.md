title: Hadoop源码 - ipc.Client
tags:
  - Hadoop
  - RPC
id: 384
categories:
  - 技术分享
date: 2012-02-25 23:06:42
---

### 1、前言

上篇博客分析了ipc包下的RPC类，这篇博客来看看Client类吧。

<!--more-->

Hadoop的RPC机制还是挺简单的，简述如下：

1. 创建代理对象；
2. 代理对象调用相应方法（invoke()）；
3. invoke调用client对象的call方法，向服务器发送请求（参数、方法）；
4. 再等待call方法的完成；
5. 返回请求结果。

具体是怎么实现的，下面来看看吧。

### 2、Client类分析

有了RPC大致分析，Client我就挑重要的分析了。

**1）connections**

RPC中就有了ClientCache类，那么client可以复用，所以一个client对象会有多个连接对象，实现中是用`HashTable<ConnectionId, Connection>`存储的连接对象。

其中，ConnectionId是用来唯一标识连接的ID，主要由客户端地址、时间戳再结合一个素数生成；Connection是Client中的静态内部类，用以处理远程连接对象。

**2）Call**

客户端方法调用的实体类，存放了id、参数、返回值等。需要注意的是，Call类中有callComplete()方法，在一次call调用完毕之后调用，并调用notify()通知client接收完毕。

**3）`call(Writable param, ConnectionId remoteId)`**

``` java
public Object invoke(Object proxy, Method method, Object[] args)  
      throws Throwable {  
      …  

      ObjectWritable value = (ObjectWritable)  
        client.call(new Invocation(method, args), remoteId); // 调用方法  
      …  
      return value.get();  
}  
```

``` java
// client请求方法  
public Writable call(Writable param, ConnectionId remoteId)    
                       throws InterruptedException, IOException {  
    Call call = new Call(param);  
    Connection connection = getConnection(remoteId, call); // 获得连接对象  
    connection.sendParam(call);                 // 发送参数  
    …  
    synchronized (call) {  
      while (!call.done) {  
        try {  
          call.wait();                           // 等待response  
        } catch (InterruptedException ie) {  
          interrupted = true;  
        }  
      }  
      …  
      return call.value;  
      …  
    }  
}
```

``` java
// Connection线程，等待服务器响应  
public void run() {  
      if (LOG.isDebugEnabled())  
        LOG.debug(getName() + ": starting, having connections "   
            + connections.size());  

      while (waitForWork()) {// 等待工作，主要依据是calls是否为空  
        receiveResponse();  // 接收响应  
      }  

      close();  

      …  
}  
```

``` java
private void receiveResponse() {  
      …  

      try {  
        int id = in.readInt();                    // try to read an id  
        …  
        Call call = calls.get(id);  

        int state = in.readInt();     // read call status  
        if (state == Status.SUCCESS.state) {  
          Writable value = ReflectionUtils.newInstance(valueClass, conf);  
          value.readFields(in);                 // read value  
          call.setValue(value);  
          calls.remove(id);  
        }   
        …  
      } catch (IOException e) {  
        markClosed(e);  
      }  
}  
```

``` java
public synchronized void setValue(Writable value) {  
      this.value = value;  
      callComplete();  
}  
protected synchronized void callComplete() {  
      this.done = true;  
      notify();  // notify caller  
}  
```

该方法由invoker调用，调用过程如下：

1. 构建Call对象
2. 用remoteId获得connection对象
   2.1 如果connections中有remoteId，取得该connection；反之，创建一个，并添加进connections；
   2.2 connection.setupIOstreams()连接到服务器，并配置好连接对象，发送协议头，接着运行connection线程，等待接收工作（waitForWork()）；
3. 调用connection.sendParam发送协议体，等待接收响应，call.wait();；
   3.1 receiveResponse()，依次读入call的值（id，value）；
   3.2 标记接收结束（markClosed），同时notifyAll()；
4. 获得返回值，返回调用者。

### 3、异步/同步模型

Hadoop的RPC对外的接口其实是同步的，但是，RPC的内部实现其实是异步消息机制。hadoop用线程wait/notify机制实现异步转同步，发送请求（call）之后wait请求处理完毕，接收完响应（connection.receiveResponse()）之后notify，notify()方法在call.setValue中。

但现在有一个问题，一个connection有多个call。可能同时有多个call在等待接收消息，那么是当client接收到response后，怎样确认它到底是之前哪个request的response呢？这个就是依靠的connection中的一个HashTable&lt;Integer, Call&gt;了，其中的Integer是用来标识Call，这样就可以将request和response对应上了。

> **参考资料：**
>
> [智障大师的专栏](http://blog.csdn.net/historyasamirror/article/details/6159248)
