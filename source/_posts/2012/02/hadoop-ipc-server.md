title: Hadoop源码 - ipc.Server
tags:
  - Hadoop
  - RPC
id: 387
categories:
  - 技术分享
date: 2012-02-27 12:57:25
---

### 1、前言

昨天分析了ipc包下的RPC、Client类，今天来分析下ipc.Server。Server类因为是Hadoop自己使用，所以代码结构以及流程都很清晰，可以清楚的看到实例化、停止、运行等过程。

<!--more-->

### 2、Server类结构

上面是Server的五个内部类，分别介绍一下：

1）Call

用以存储客户端发来的请求，这个请求会放入一个BlockQueue中；

2）Listener

监听类，用以监听客户端发来的请求。同时Listener下面还有一个静态类，Listener.Reader，当监听器监听到用户请求，便用让Reader读取用户请求。

3）Responder

响应RPC请求类，请求处理完毕，由Responder发送给请求客户端。

4）Connection

连接类，真正的客户端请求读取逻辑在这个类中。

5）Handler

请求（blockQueueCall）处理类，会循环阻塞读取callQueue中的call对象，并对其进行操作。

3、Server初始化

第一篇[博客](http://www.hongweiyi.com/2012/02/hadoop-ipc-rpc/)说了，Server的初始化入口在RPC.getServer中，getServer其实是调用的RPC.Server静态类中的构造方法，我们看看Namenode创建RPCServer的方法和RPC.Server构造方法代码：

``` java
private void initialize(Configuration conf) throws IOException {  
    …  
    this.serviceRpcServer = RPC.getServer(this, dnSocketAddr.getHostName(),   
          dnSocketAddr.getPort(), serviceHandlerCount,  
          false, conf, namesystem.getDelegationTokenSecretManager());  
    this.serviceRpcServer.start(); // 运行服务器  
}  
```         

``` java
public Server(Object instance, Configuration conf, String bindAddress,  int port,  
                  int numHandlers, boolean verbose,   
                  SecretManager<? extends TokenIdentifier> secretManager)   
        throws IOException {  
      super(bindAddress, port, Invocation.class, numHandlers, conf,  
          classNameBase(instance.getClass().getName()), secretManager);  
      this.instance = instance;  
      this.verbose = verbose;  
}  
```

该方法调用了父类的构造方法，如下：

``` java
protected Server(String bindAddress, int port,   
                  Class<? extends Writable> paramClass, int handlerCount,   
                  Configuration conf, String serverName, SecretManager<? extends TokenIdentifier> secretManager)   
    throws IOException {  
    this.bindAddress = bindAddress;  
    this.conf = conf;  
    this.port = port;  
    this.paramClass = paramClass;  
    this.handlerCount = handlerCount;  
    this.socketSendBufferSize = 0;  
    this.maxQueueSize = handlerCount * conf.getInt(  
                                IPC_SERVER_HANDLER_QUEUE_SIZE_KEY,  
                                IPC_SERVER_HANDLER_QUEUE_SIZE_DEFAULT);  
    this.maxRespSize = conf.getInt(IPC_SERVER_RPC_MAX_RESPONSE_SIZE_KEY,  
                                   IPC_SERVER_RPC_MAX_RESPONSE_SIZE_DEFAULT);  
    this.readThreads = conf.getInt(  
        IPC_SERVER_RPC_READ_THREADS_KEY,  
        IPC_SERVER_RPC_READ_THREADS_DEFAULT);  
    this.callQueue  = new LinkedBlockingQueue<Call>(maxQueueSize);   
    this.maxIdleTime = 2*conf.getInt("ipc.client.connection.maxidletime", 1000);  
    this.maxConnectionsToNuke = conf.getInt("ipc.client.kill.max", 10);  
    this.thresholdIdleConnections = conf.getInt("ipc.client.idlethreshold", 4000);  
    this.secretManager = (SecretManager<TokenIdentifier>) secretManager;  
    this.authorize =   
      conf.getBoolean(HADOOP_SECURITY_AUTHORIZATION, false);  
    this.isSecurityEnabled = UserGroupInformation.isSecurityEnabled();  

    // Start the listener here and let it bind to the port  
    listener = new Listener();  
    this.port = listener.getAddress().getPort();      
    this.rpcMetrics = RpcInstrumentation.create(serverName, this.port);  
    this.tcpNoDelay = conf.getBoolean("ipc.server.tcpnodelay", false);  

    // Create the responder here  
    responder = new Responder();  

    if (isSecurityEnabled) {  
      SaslRpcServer.init(conf);  
    }  
}  
```

不难看出，父类的构造方法就初始化了一些配置和变量。

### 4、Server运行

在上面第一段代码中，还有一句RpcServer.start()的方法，在调用构造函数初始化一些变量之后，Server就可以正式运行起来了：

``` java
public synchronized void start() {  
    responder.start();  
    listener.start();  
    handlers = new Handler[handlerCount];  

    for (int i = 0; i < handlerCount; i++) {  
      handlers[i] = new Handler(i);  
      handlers[i].start();  
    }  
}
```

responder、listener、handlers三个对象的线程均阻塞了，前两个阻塞在selector.select()方法上，handler阻塞在callQueue.take()方法，都在等待客户端请求。Responder设置了超时时间，为15分钟。而listener还开启了Reader线程，该线程也阻塞了。

### 4、Server接受请求流程

1）监听到请求

Listener监听到请求，获得所有请求的SelectionKey，执行doAccept(key)方法，该方法将所有的连接对象放入list中，并将connection对象与key绑定，以供reader使用。初始化玩所有的conne对象之后，就可以激活Reader线程了。

``` java
void doAccept(SelectionKey key) throws IOException,  OutOfMemoryError {  
      Connection c = null;  
      ServerSocketChannel server = (ServerSocketChannel) key.channel();  
      SocketChannel channel;  
      while ((channel = server.accept()) != null) {  
        channel.configureBlocking(false);  
        channel.socket().setTcpNoDelay(tcpNoDelay);  
        Reader reader = getReader();  
        try {  
          reader.startAdd();  // 激活readSelector，设置adding为true  
          SelectionKey readKey = reader.registerChannel(channel);  
          c = new Connection(readKey, channel, System.currentTimeMillis());  
          readKey.attach(c);  
          synchronized (connectionList) {  
            connectionList.add(numConnections, c);  
            numConnections++;  
          }  
          …           
        } finally {  
          reader.finishAdd(); // add完毕，设置adding为false，Reader开始工作  
        }  
    }  
}
```

2）接收请求

Reader的run方法和Listener基本一致，也是获得所有的SelectionKey，再执行doRead(key)方法。该方法获得key中绑定的connection，并执行conection的readAndProcess()方法：

``` java
void doRead(SelectionKey key) throws InterruptedException {  
      int count = 0;  
      Connection c = (Connection)key.attachment(); // 获得连接对象  
      if (c == null) {  
        return;    
      }  
      c.setLastContact(System.currentTimeMillis());  

      try {  
        count = c.readAndProcess(); // 接受并处理请求  
      } catch (InterruptedException ieo) {  
        …  
      }  
      if (count < 0) {  
        …  
        closeConnection(c);  
        c = null;  
      }  
      else {  
        c.setLastContact(System.currentTimeMillis());  
      }  
}  
```            

``` java
public int readAndProcess() throws IOException, InterruptedException {  
    // 一次最多读取一次RPC请求，如果头没读完，继续迭代直到  
    // 读完所有请求数据    
    while (true) {   
        int count = –1;  
        if (dataLengthBuffer.remaining() > 0) {  
          count = channelRead(channel, dataLengthBuffer);         
          …  
        if (!rpcHeaderRead) {  
          //读取请求头.  
          if (rpcHeaderBuffer == null) {  
            rpcHeaderBuffer = ByteBuffer.allocate(2);  
          }  
          count = channelRead(channel, rpcHeaderBuffer);  
          if (count < 0 || rpcHeaderBuffer.remaining() > 0) {  
            return count;  
          }  
          // 读取请求版本号  
          int version = rpcHeaderBuffer.get(0);  
          byte[] method = new byte[] {rpcHeaderBuffer.get(1)};  
          authMethod = AuthMethod.read(new DataInputStream(  
              new ByteArrayInputStream(method)));  
          dataLengthBuffer.flip();            
          …  
          dataLengthBuffer.clear();  
          …  

          rpcHeaderBuffer = null;  
          rpcHeaderRead = true;  
          continue;  
        }   
        …  
          data = ByteBuffer.allocate(dataLength);  
        }  

        // 读取请求  
        count = channelRead(channel, data);  

        if (data.remaining() == 0) {  
          …  
          if (useSasl) {  
            saslReadAndProcess(data.array());  
          } else {  
            // 执行RPC请求，先解析header请求，下次循环解析param请求  
            processOneRpc(data.array());  
          }  
          …  
        }   
        return count;  
    }  
}  
```

3）获得call请求

在Connection中解析param请求中，解析了请求数据，并构造Call对象，将其加入callQueue。

``` java
private void processData(byte[] buf) throws  IOException, InterruptedException {  
      DataInputStream dis =  
        new DataInputStream(new ByteArrayInputStream(buf));  
      int id = dis.readInt();         // 读取请求id  
        …  

      Writable param = ReflectionUtils.newInstance(paramClass, conf);// 获取参数，paramClass是参数的实体类，在构造Server对象的时候传入  
      param.readFields(dis);          

      Call call = new Call(id, param, this);  
      callQueue.put(call);              // 添加进阻塞队列，不过队列有max限制，有可能也会阻塞  
      incRpcCount();   
}  
```

4）处理call对象

Connection给callQueue添加了call对象，阻塞的Handler可以继续运行了，拿出一个call对象，并调用RPC.Call方法。

``` java
// 关键代码  
while (running) {  
   final Call call = callQueue.take(); // 弹出call对象  
   CurCall.set(call);  
   value = call(call.connection.protocol, call.param,   
                         call.timestamp); // 调用RPC.Server中的call  
   CurCall.set(null);  

   synchronized (call.connection.responseQueue) {  
       setupResponse(buf, call,   
                      (error == null) ? Status.SUCCESS : Status.ERROR,   
                      value, errorClass, error);  
       …  
       responder.doRespond(call);  
   }  
}  
```

5）响应请求

上面代码中的setupResponse将call的id和状态发送回去，再设置了call中的response:ByteBuffer，之后就开始responder.doRespond(call)了，processResponse以及Responder.run()没太弄明白，就先不说了。

``` java
void doRespond(Call call) throws IOException {  
  synchronized (call.connection.responseQueue) {  
    // 这行没懂  
    call.connection.responseQueue.addLast(call);  
    if (call.connection.responseQueue.size() == 1) {  
      // 返回响应结果，并激活writeSelector  
      processResponse(call.connection.responseQueue, true);  
    }  
  }  
}  
  ```

### 6、总结

Server用的标准的Java TCP/IP NIO通信，同时请求的超时使用基于BlockingQueue以及wait/notify机制实现。使用的模式是reactor模式，关于nio和reactor可以参考这个[博客](http://www.cnblogs.com/ericchen/archive/2011/05/08/2036993.html)。

对于服务器端接收多个连接请求的需求，Server采用Listener来监听连接的事件，并用Listener.Reader来监听网络流读以及Responder监听写的事件，当有实际的网络流读写时间发生之后，解析了请求Call之后，添加进阻塞队列，并交由多个Handlers来处理请求。

这个方法比TCP/IP BIO好处就是可接受很多的连接，而这些连接只在真实的请求时才会创建线程处理，称之为一请求一处理。但是，连接上的请求发送非常频繁时，TCP/IP NIO的方法并不会带来太大的优势。

但是Hadoop实际场景中，通常是服务器端支持大量的连接数（Namenode连上几千个Datanode），但是连接发送的请求并不会太多（heartbeat、blockreport都有较长间隔）。这样就造成了Hadoop不适合实时的、多请求的运算，带来的代价是模型、实现简单，但是这也为以后的扩展埋下了祸根。

P.S.: 以上分析基于稳定版0.20.203.0rc1。
