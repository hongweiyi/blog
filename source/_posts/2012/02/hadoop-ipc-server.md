title: Hadoop源码 - ipc.Server
tags:
  - Hadoop
  - RPC
id: 387
categories:
  - 技术分享
date: 2012-02-27 12:57:25
---

**1****、前言**

昨天分析了ipc包下的RPC、Client类，今天来分析下ipc.Server。Server类因为是Hadoop自己使用，所以代码结构以及流程都很清晰，可以清楚的看到实例化、停止、运行等过程。

<!--more-->

**2****、Server类结构**

上面是Server的五个内部类，分别介绍一下：

**1****）Call**

用以存储客户端发来的请求，这个请求会放入一个BlockQueue中；

**2****）Listener**

监听类，用以监听客户端发来的请求。同时Listener下面还有一个静态类，Listener.Reader，当监听器监听到用户请求，便用让Reader读取用户请求。

**3****）Responder**

响应RPC请求类，请求处理完毕，由Responder发送给请求客户端。

**4****）Connection**

连接类，真正的客户端请求读取逻辑在这个类中。

**5****）Handler**

请求（blockQueueCall）处理类，会循环阻塞读取callQueue中的call对象，并对其进行操作。

**3****、Server初始化**

第一篇[博客](http://www.hongweiyi.com/2012/02/hadoop-ipc-rpc/)说了，Server的初始化入口在RPC.getServer中，getServer其实是调用的RPC.Server静态类中的构造方法，我们看看Namenode创建RPCServer的方法和RPC.Server构造方法代码：     <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">private</span><span>&#160;</span><span class="keyword">void</span><span> initialize(Configuration conf) </span><span class="keyword">throws</span><span> IOException {&#160;&#160; </span></span>
2.  <span>&#160;&#160;&#160; …&#160;&#160; </span>
3.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.serviceRpcServer = RPC.getServer(</span><span class="keyword">this</span><span>, dnSocketAddr.getHostName(),&#160;&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; dnSocketAddr.getPort(), serviceHandlerCount,&#160;&#160; </span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">false</span><span>, conf, namesystem.getDelegationTokenSecretManager());&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.serviceRpcServer.start(); </span><span class="comment">// 运行服务器 </span><span>&#160; </span></span>
7.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>        <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">public</span><span> Server(Object instance, Configuration conf, String bindAddress,&#160; </span><span class="keyword">int</span><span> port,&#160;&#160; </span></span>
2.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">int</span><span> numHandlers, </span><span class="keyword">boolean</span><span> verbose,&#160;&#160;&#160; </span></span>
3.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; SecretManager&lt;? </span><span class="keyword">extends</span><span> TokenIdentifier&gt; secretManager)&#160;&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">throws</span><span> IOException {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">super</span><span>(bindAddress, port, Invocation.</span><span class="keyword">class</span><span>, numHandlers, conf,&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; classNameBase(instance.getClass().getName()), secretManager);&#160;&#160; </span>
7.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">this</span><span>.instance = instance;&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">this</span><span>.verbose = verbose;&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160; }&#160;&#160; </span>           </div>            <p>
         </td>       </tr>     </tbody></table> </p>  

该方法调用了父类的构造方法，如下：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">protected</span><span> Server(String bindAddress, </span><span class="keyword">int</span><span> port,&#160;&#160;&#160; </span></span>
2.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Class&lt;? </span><span class="keyword">extends</span><span> Writable&gt; paramClass, </span><span class="keyword">int</span><span> handlerCount,&#160;&#160;&#160; </span></span>
3.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Configuration conf, String serverName, SecretManager&lt;? </span><span class="keyword">extends</span><span> TokenIdentifier&gt; secretManager)&#160;&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">throws</span><span> IOException {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.bindAddress = bindAddress;&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.conf = conf;&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.port = port;&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.paramClass = paramClass;&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.handlerCount = handlerCount;&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.socketSendBufferSize = </span><span class="number">0</span><span>;&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.maxQueueSize = handlerCount * conf.getInt(&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; IPC_SERVER_HANDLER_QUEUE_SIZE_KEY,&#160;&#160; </span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; IPC_SERVER_HANDLER_QUEUE_SIZE_DEFAULT);&#160;&#160; </span>
14.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.maxRespSize = conf.getInt(IPC_SERVER_RPC_MAX_RESPONSE_SIZE_KEY,&#160;&#160; </span></span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; IPC_SERVER_RPC_MAX_RESPONSE_SIZE_DEFAULT);&#160;&#160; </span>
16.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.readThreads = conf.getInt(&#160;&#160; </span></span>
17.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; IPC_SERVER_RPC_READ_THREADS_KEY,&#160;&#160; </span>
18.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; IPC_SERVER_RPC_READ_THREADS_DEFAULT);&#160;&#160; </span>
19.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.callQueue&#160; = </span><span class="keyword">new</span><span> LinkedBlockingQueue&lt;Call&gt;(maxQueueSize);&#160;&#160;&#160; </span></span>
20.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.maxIdleTime = </span><span class="number">2</span><span>*conf.getInt(</span><span class="string">&quot;ipc.client.connection.maxidletime&quot;</span><span>, </span><span class="number">1000</span><span>);&#160;&#160; </span></span>
21.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.maxConnectionsToNuke = conf.getInt(</span><span class="string">&quot;ipc.client.kill.max&quot;</span><span>, </span><span class="number">10</span><span>);&#160;&#160; </span></span>
22.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.thresholdIdleConnections = conf.getInt(</span><span class="string">&quot;ipc.client.idlethreshold&quot;</span><span>, </span><span class="number">4000</span><span>);&#160;&#160; </span></span>
23.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.secretManager = (SecretManager&lt;TokenIdentifier&gt;) secretManager;&#160;&#160; </span></span>
24.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.authorize =&#160;&#160;&#160; </span></span>
25.  <span>&#160;&#160;&#160;&#160;&#160; conf.getBoolean(HADOOP_SECURITY_AUTHORIZATION, </span><span class="keyword">false</span><span>);&#160;&#160; </span></span>
26.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.isSecurityEnabled = UserGroupInformation.isSecurityEnabled();&#160;&#160; </span></span>
27.  <span>&#160;&#160;&#160;&#160;&#160;&#160; </span>
28.  <span>&#160;&#160;&#160; </span><span class="comment">// Start the listener here and let it bind to the port </span><span>&#160; </span></span>
29.  <span>&#160;&#160;&#160; listener = </span><span class="keyword">new</span><span> Listener();&#160;&#160; </span></span>
30.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.port = listener.getAddress().getPort();&#160;&#160;&#160;&#160;&#160;&#160; </span></span>
31.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.rpcMetrics = RpcInstrumentation.create(serverName, </span><span class="keyword">this</span><span>.port);&#160;&#160; </span></span>
32.  <span>&#160;&#160;&#160; </span><span class="keyword">this</span><span>.tcpNoDelay = conf.getBoolean(</span><span class="string">&quot;ipc.server.tcpnodelay&quot;</span><span>, </span><span class="keyword">false</span><span>);&#160;&#160; </span></span>
33.  <span>&#160; </span>
34.  <span>&#160;&#160;&#160; </span><span class="comment">// Create the responder here </span><span>&#160; </span></span>
35.  <span>&#160;&#160;&#160; responder = </span><span class="keyword">new</span><span> Responder();&#160;&#160; </span></span>
36.  <span>&#160;&#160;&#160;&#160;&#160;&#160; </span>
37.  <span>&#160;&#160;&#160; </span><span class="keyword">if</span><span> (isSecurityEnabled) {&#160;&#160; </span></span>
38.  <span>&#160;&#160;&#160;&#160;&#160; SaslRpcServer.init(conf);&#160;&#160; </span>
39.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
40.  <span>&#160; }&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

不难看出，父类的构造方法就初始化了一些配置和变量。

**4、Server运行**

在上面第一段代码中，还有一句RpcServer.start()的方法，在调用构造函数初始化一些变量之后，Server就可以正式运行起来了：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">public</span><span>&#160;</span><span class="keyword">synchronized</span><span>&#160;</span><span class="keyword">void</span><span> start() {&#160;&#160; </span></span>
2.  <span>&#160;&#160;&#160; responder.start();&#160;&#160; </span>
3.  <span>&#160;&#160;&#160; listener.start();&#160;&#160; </span>
4.  <span>&#160;&#160;&#160; handlers = </span><span class="keyword">new</span><span> Handler[handlerCount];&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160; </span>
6.  <span>&#160;&#160;&#160; </span><span class="keyword">for</span><span> (</span><span class="keyword">int</span><span> i = </span><span class="number">0</span><span>; i &lt; handlerCount; i++) {&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160; handlers[i] = </span><span class="keyword">new</span><span> Handler(i);&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160; handlers[i].start();&#160;&#160; </span>
9.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
10.  <span>&#160; }&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

responder、listener、handlers三个对象的线程均阻塞了，前两个阻塞在selector.select()方法上，handler阻塞在callQueue.take()方法，都在等待客户端请求。Responder设置了超时时间，为15分钟。而listener还开启了Reader线程，该线程也阻塞了。

**4、Server接受请求流程**

**1****）监听到请求**

Listener监听到请求，获得所有请求的SelectionKey，执行doAccept(key)方法，该方法将所有的连接对象放入list中，并将connection对象与key绑定，以供reader使用。初始化玩所有的conne对象之后，就可以激活Reader线程了。    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">void</span><span> doAccept(SelectionKey key) </span><span class="keyword">throws</span><span> IOException,&#160; OutOfMemoryError {&#160;&#160; </span></span>
2.  <span>&#160;&#160;&#160;&#160;&#160; Connection c = </span><span class="keyword">null</span><span>;&#160;&#160; </span></span>
3.  <span>&#160;&#160;&#160;&#160;&#160; ServerSocketChannel server = (ServerSocketChannel) key.channel();&#160;&#160; </span>
4.  <span>&#160;&#160;&#160;&#160;&#160; SocketChannel channel;&#160;&#160; </span>
5.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">while</span><span> ((channel = server.accept()) != </span><span class="keyword">null</span><span>) {&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; channel.configureBlocking(</span><span class="keyword">false</span><span>);&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; channel.socket().setTcpNoDelay(tcpNoDelay);&#160;&#160; </span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Reader reader = getReader();&#160;&#160; </span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">try</span><span> {&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; reader.startAdd();&#160; </span><span class="comment">// 激活readSelector，设置adding为true </span><span>&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; SelectionKey readKey = reader.registerChannel(channel);&#160;&#160; </span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; c = </span><span class="keyword">new</span><span> Connection(readKey, channel, System.currentTimeMillis());&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; readKey.attach(c);&#160;&#160; </span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">synchronized</span><span> (connectionList) {&#160;&#160; </span></span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; connectionList.add(numConnections, c);&#160;&#160; </span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; numConnections++;&#160;&#160; </span>
17.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
18.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
19.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">finally</span><span> {&#160;&#160; </span></span>
20.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; reader.finishAdd(); </span><span class="comment">// add完毕，设置adding为false，Reader开始工作 </span><span>&#160; </span></span>
21.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
22.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
23.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

**2****）接收请求**

Reader的run方法和Listener基本一致，也是获得所有的SelectionKey，再执行doRead(key)方法。该方法获得key中绑定的connection，并执行conection的readAndProcess()方法：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">void</span><span> doRead(SelectionKey key) </span><span class="keyword">throws</span><span> InterruptedException {&#160;&#160; </span></span>
2.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">int</span><span> count = </span><span class="number">0</span><span>;&#160;&#160; </span></span>
3.  <span>&#160;&#160;&#160;&#160;&#160; Connection c = (Connection)key.attachment(); </span><span class="comment">// 获得连接对象 </span><span>&#160; </span></span>
4.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (c == </span><span class="keyword">null</span><span>) {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span>;&#160;&#160;&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
7.  <span>&#160;&#160;&#160;&#160;&#160; c.setLastContact(System.currentTimeMillis());&#160;&#160; </span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
9.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">try</span><span> {&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; count = c.readAndProcess(); </span><span class="comment">// 接受并处理请求 </span><span>&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">catch</span><span> (InterruptedException ieo) {&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
13.  <span>&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
14.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (count &lt; </span><span class="number">0</span><span>) {&#160;&#160; </span></span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; closeConnection(c);&#160;&#160; </span>
17.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; c = </span><span class="keyword">null</span><span>;&#160;&#160; </span></span>
18.  <span>&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
19.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">else</span><span> {&#160;&#160; </span></span>
20.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; c.setLastContact(System.currentTimeMillis());&#160;&#160; </span>
21.  <span>&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
22.  <span>&#160;&#160;&#160; }&#160;&#160; </span>           </div>         </td>       </tr>        <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">public</span><span>&#160;</span><span class="keyword">int</span><span> readAndProcess() </span><span class="keyword">throws</span><span> IOException, InterruptedException {&#160;&#160; </span></span>
2.  <span></span><span class="comment">// 一次最多读取一次RPC请求，如果头没读完，继续迭代直到 </span><span>&#160; </span></span>
3.  <span></span><span class="comment">// 读完所有请求数据&#160;&#160; </span><span>&#160; </span></span>
4.  <span></span><span class="keyword">while</span><span> (</span><span class="keyword">true</span><span>) {&#160;&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">int</span><span> count = -</span><span class="number">1</span><span>;&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (dataLengthBuffer.remaining() &gt; </span><span class="number">0</span><span>) {&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; count = channelRead(channel, dataLengthBuffer);&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (!rpcHeaderRead) {&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">//读取请求头. </span><span>&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (rpcHeaderBuffer == </span><span class="keyword">null</span><span>) {&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; rpcHeaderBuffer = ByteBuffer.allocate(</span><span class="number">2</span><span>);&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; count = channelRead(channel, rpcHeaderBuffer);&#160;&#160; </span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (count &lt; </span><span class="number">0</span><span> || rpcHeaderBuffer.remaining() &gt; </span><span class="number">0</span><span>) {&#160;&#160; </span></span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> count;&#160;&#160; </span></span>
17.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
18.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 读取请求版本号 </span><span>&#160; </span></span>
19.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">int</span><span> version = rpcHeaderBuffer.get(</span><span class="number">0</span><span>);&#160;&#160; </span></span>
20.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">byte</span><span>[] method = </span><span class="keyword">new</span><span>&#160;</span><span class="keyword">byte</span><span>[] {rpcHeaderBuffer.get(</span><span class="number">1</span><span>)};&#160;&#160; </span></span>
21.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; authMethod = AuthMethod.read(</span><span class="keyword">new</span><span> DataInputStream(&#160;&#160; </span></span>
22.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">new</span><span> ByteArrayInputStream(method)));&#160;&#160; </span></span>
23.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; dataLengthBuffer.flip();&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
24.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
25.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; dataLengthBuffer.clear();&#160;&#160; </span>
26.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
27.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
28.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; rpcHeaderBuffer = </span><span class="keyword">null</span><span>;&#160;&#160; </span></span>
29.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; rpcHeaderRead = </span><span class="keyword">true</span><span>;&#160;&#160; </span></span>
30.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">continue</span><span>;&#160;&#160; </span></span>
31.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160;&#160; </span>
32.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
33.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; data = ByteBuffer.allocate(dataLength);&#160;&#160; </span>
34.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
35.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
36.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 读取请求 </span><span>&#160; </span></span>
37.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; count = channelRead(channel, data);&#160;&#160; </span>
38.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
39.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (data.remaining() == </span><span class="number">0</span><span>) {&#160;&#160; </span></span>
40.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
41.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (useSasl) {&#160;&#160; </span></span>
42.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; saslReadAndProcess(data.array());&#160;&#160; </span>
43.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span> {&#160;&#160; </span></span>
44.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 执行RPC请求，先解析header请求，下次循环解析param请求 </span><span>&#160; </span></span>
45.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; processOneRpc(data.array());&#160;&#160; </span>
46.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
47.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
48.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160;&#160; </span>
49.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> count;&#160;&#160; </span></span>
50.  <span>&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
51.  <span>&#160;&#160;&#160; }&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

**3****）获得call请求**

在Connection中解析param请求中，解析了请求数据，并构造Call对象，将其加入callQueue。    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">private</span><span>&#160;</span><span class="keyword">void</span><span> processData(</span><span class="keyword">byte</span><span>[] buf) </span><span class="keyword">throws</span><span>&#160; IOException, InterruptedException {&#160;&#160; </span></span>
2.  <span>&#160;&#160;&#160;&#160;&#160; DataInputStream dis =&#160;&#160; </span>
3.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">new</span><span> DataInputStream(</span><span class="keyword">new</span><span> ByteArrayInputStream(buf));&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">int</span><span> id = dis.readInt();&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 读取请求id </span><span>&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
6.  <span>&#160; </span>
7.  <span>&#160;&#160;&#160;&#160;&#160; Writable param = ReflectionUtils.newInstance(paramClass, conf);</span><span class="comment">// 获取参数，paramClass是参数的实体类，在构造Server对象的时候传入 </span><span>&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160; param.readFields(dis);&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
10.  <span>&#160;&#160;&#160;&#160;&#160; Call call = </span><span class="keyword">new</span><span> Call(id, param, </span><span class="keyword">this</span><span>);&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160; callQueue.put(call);&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 添加进阻塞队列，不过队列有max限制，有可能也会阻塞 </span><span>&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160; incRpcCount();&#160;&#160;&#160; </span>
13.  <span>&#160;&#160;&#160; }&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

**4****）处理call对象**

Connection给callQueue添加了call对象，阻塞的Handler可以继续运行了，拿出一个call对象，并调用RPC.Call方法    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="comment">// 关键代码 </span><span>&#160; </span></span>
2.  <span></span><span class="keyword">while</span><span> (running) {&#160;&#160; </span></span>
3.  <span></span><span class="keyword">final</span><span> Call call = callQueue.take(); </span><span class="comment">// 弹出call对象 </span><span>&#160; </span></span>
4.  <span>CurCall.set(call);&#160;&#160; </span>
5.  <span>&#160;&#160;&#160;&#160; value = call(call.connection.protocol, call.param,&#160;&#160;&#160; </span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; call.timestamp); </span><span class="comment">// 调用RPC.Server中的call </span><span>&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160; CurCall.set(</span><span class="keyword">null</span><span>);&#160;&#160; </span></span>
8.  <span>&#160; </span>
9.  <span></span><span class="keyword">synchronized</span><span> (call.connection.responseQueue) {&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; setupResponse(buf, call,&#160;&#160;&#160; </span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; (error == </span><span class="keyword">null</span><span>) ? Status.SUCCESS : Status.ERROR,&#160;&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; value, errorClass, error);&#160;&#160; </span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; responder.doRespond(call);&#160;&#160; </span>
15.  <span>&#160;&#160;&#160;&#160; }&#160;&#160; </span>
16.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

**5）响应请求**

上面代码中的setupResponse将call的id和状态发送回去，再设置了call中的response:ByteBuffer，之后就开始responder.doRespond(call)了，processResponse以及Responder.run()没太弄明白，就先不说了。    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">void</span><span> doRespond(Call call) </span><span class="keyword">throws</span><span> IOException {&#160;&#160; </span></span>2.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">synchronized</span><span> (call.connection.responseQueue) {&#160;&#160; </span></span>3.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 这行没懂 </span><span>&#160; </span></span>4.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; call.connection.responseQueue.addLast(call);&#160;&#160; </span>5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (call.connection.responseQueue.size() == </span><span class="number">1</span><span>) {&#160;&#160; </span></span>6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 返回响应结果，并激活writeSelector </span><span>&#160; </span></span>7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; processResponse(call.connection.responseQueue, </span><span class="keyword">true</span><span>);&#160;&#160; </span></span>8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>9.  <span>&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>10.  <span>&#160;&#160;&#160; }&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

**6、总结**

Server用的标准的Java TCP/IP NIO通信，同时请求的超时使用基于BlockingQueue以及wait/notify机制实现。使用的模式是reactor模式，关于nio和reactor可以参考这个**[博客](http://www.cnblogs.com/ericchen/archive/2011/05/08/2036993.html)**。

对于服务器端接收多个连接请求的需求，Server采用Listener来监听连接的事件，并用Listener.Reader来监听网络流读以及Responder监听写的事件，当有实际的网络流读写时间发生之后，解析了请求Call之后，添加进阻塞队列，并交由多个Handlers来处理请求。

这个方法比TCP/IP BIO好处就是可接受很多的连接，而这些连接只在真实的请求时才会创建线程处理，称之为一请求一处理。但是，连接上的请求发送非常频繁时，TCP/IP NIO的方法并不会带来太大的优势。

但是Hadoop实际场景中，通常是服务器端支持大量的连接数（Namenode连上几千个Datanode），但是连接发送的请求并不会太多（heartbeat、blockreport都有较长间隔）。这样就造成了Hadoop不适合实时的、多请求的运算，带来的代价是模型、实现简单，但是这也为以后的扩展埋下了祸根。

P.S.: 以上分析基于稳定版0.20.203.0rc1。
