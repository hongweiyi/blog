title: Hadoop源码 - ipc.Client
tags:
  - Hadoop
  - RPC
id: 384
categories:
  - 技术分享
date: 2012-02-25 23:06:42
---

**1****、前言**

上篇博客分析了ipc包下的RPC类，这篇博客来看看Client类吧。
  <!--more-->

Hadoop的RPC机制还是挺简单的，简述如下：

1）创建代理对象；

2）代理对象调用相应方法（invoke()）；

3）invoke调用client对象的call方法，向服务器发送请求（参数、方法）；

4）再等待call方法的完成；

5）返回请求结果。

具体是怎么实现的，下面来看看吧。

**2****、Client类分析**

有了RPC大致分析，Client我就挑重要的分析了。

**1）connections**

RPC中就有了ClientCache类，那么client可以复用，所以一个client对象会有多个连接对象，实现中是用HashTable&lt;ConnectionId, Connection&gt;存储的连接对象。

其中，ConnectionId是用来唯一标识连接的ID，主要由客户端地址、时间戳再结合一个素数生成；Connection是Client中的静态内部类，用以处理远程连接对象。

**2）Call**

客户端方法调用的实体类，存放了id、参数、返回值等。需要注意的是，Call类中有callComplete()方法，在一次call调用完毕之后调用，并调用notify()通知client接收完毕。

**3）call(Writable param, ConnectionId remoteId)**     <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">public</span><span> Object invoke(Object proxy, Method method, Object[] args)&#160;&#160; </span></span>
2.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">throws</span><span> Throwable {&#160;&#160; </span></span>
3.  <span>&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
4.  <span>&#160; </span>
5.  <span>&#160;&#160;&#160;&#160;&#160; ObjectWritable value = (ObjectWritable)&#160;&#160; </span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; client.call(</span><span class="keyword">new</span><span> Invocation(method, args), remoteId); </span><span class="comment">// 调用方法 </span><span>&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
8.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> value.get();&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160; }&#160;&#160; </span>           </div>         </td>       </tr>        <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="comment">// client请求方法 </span><span>&#160; </span></span>
2.  <span></span><span class="keyword">public</span><span> Writable call(Writable param, ConnectionId remoteId)&#160;&#160;&#160;&#160; </span></span>
3.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">throws</span><span> InterruptedException, IOException {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; Call call = </span><span class="keyword">new</span><span> Call(param);&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160; Connection connection = getConnection(remoteId, call); </span><span class="comment">// 获得连接对象 </span><span>&#160; </span></span>
6.  <span>&#160;&#160;&#160; connection.sendParam(call);&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 发送参数 </span><span>&#160; </span></span>
7.  <span>&#160;&#160;&#160; …&#160;&#160; </span>
8.  <span>&#160;&#160;&#160; </span><span class="keyword">synchronized</span><span> (call) {&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">while</span><span> (!call.done) {&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">try</span><span> {&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; call.wait();&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 等待response </span><span>&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">catch</span><span> (InterruptedException ie) {&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; interrupted = </span><span class="keyword">true</span><span>;&#160;&#160; </span></span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
15.  <span>&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
16.  <span>&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
17.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> call.value;&#160;&#160; </span></span>
18.  <span>&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
19.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
20.  <span>&#160; }&#160;&#160; </span>           </div>         </td>       </tr>        <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="comment">// Connection线程，等待服务器响应 </span><span>&#160; </span></span>
2.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> run() {&#160;&#160; </span></span>
3.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (LOG.isDebugEnabled())&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; LOG.debug(getName() + </span><span class="string">&quot;: starting, having connections &quot;</span><span>&#160;&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; + connections.size());&#160;&#160; </span>
6.  <span>&#160; </span>
7.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">while</span><span> (waitForWork()) {</span><span class="comment">// 等待工作，主要依据是calls是否为空 </span><span>&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; receiveResponse();&#160; </span><span class="comment">// 接收响应 </span><span>&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
11.  <span>&#160;&#160;&#160;&#160;&#160; close();&#160;&#160; </span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
13.  <span>&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
14.  <span>&#160;&#160;&#160; }&#160;&#160; </span>           </div>         </td>       </tr>        <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">private</span><span>&#160;</span><span class="keyword">void</span><span> receiveResponse() {&#160;&#160; </span></span>
2.  <span>&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
3.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
4.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">try</span><span> {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">int</span><span> id = in.readInt();&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// try to read an id </span><span>&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Call call = calls.get(id);&#160;&#160; </span>
8.  <span>&#160; </span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">int</span><span> state = in.readInt();&#160;&#160;&#160;&#160; </span><span class="comment">// read call status </span><span>&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (state == Status.SUCCESS.state) {&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; Writable value = ReflectionUtils.newInstance(valueClass, conf);&#160;&#160; </span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; value.readFields(in);&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// read value </span><span>&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; call.setValue(value);&#160;&#160; </span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; calls.remove(id);&#160;&#160; </span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160;&#160; </span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; …&#160;&#160; </span>
17.  <span>&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">catch</span><span> (IOException e) {&#160;&#160; </span></span>
18.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; markClosed(e);&#160;&#160; </span>
19.  <span>&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
20.  <span>&#160;&#160;&#160; }&#160;&#160; </span>           </div>         </td>       </tr>        <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">public</span><span>&#160;</span><span class="keyword">synchronized</span><span>&#160;</span><span class="keyword">void</span><span> setValue(Writable value) {&#160;&#160; </span></span>
2.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">this</span><span>.value = value;&#160;&#160; </span></span>
3.  <span>&#160;&#160;&#160;&#160;&#160; callComplete();&#160;&#160; </span>
4.  <span>}&#160;&#160; </span>
5.  <span></span><span class="keyword">protected</span><span>&#160;</span><span class="keyword">synchronized</span><span>&#160;</span><span class="keyword">void</span><span> callComplete() {&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">this</span><span>.done = </span><span class="keyword">true</span><span>;&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160; notify();&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// notify caller </span><span>&#160; </span></span>
8.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

该方法由invoker调用，调用过程如下：

A、构建Call对象

B、 用remoteId获得connection对象

&#160;&#160;&#160; a) 如果connections中有remoteId，取得该connection；反之，创建一个，并添加进connections；

&#160;&#160;&#160; b) connection.setupIOstreams()连接到服务器，并配置好连接对象，发送协议头，接着运行connection线程，等待接收工作（waitForWork()）；

C、 调用connection.sendParam发送协议体，等待接收响应，call.wait();；

&#160;&#160;&#160; a) receiveResponse()，依次读入call的值（id，value）；

&#160;&#160;&#160; b) 标记接收结束（markClosed），同时notifyAll()；

D、 获得返回值，返回调用者。

**3、异步/同步模型**

Hadoop的RPC对外的接口其实是同步的，但是，RPC的内部实现其实是异步消息机制。hadoop用线程wait/notify机制实现异步转同步，发送请求（call）之后wait请求处理完毕，接收完响应（connection.receiveResponse()）之后notify，notify()方法在call.setValue中。

但现在有一个问题，一个connection有多个call。可能同时有多个call在等待接收消息，那么是当client接收到response后，怎样确认它到底是之前哪个request的response呢？这个就是依靠的connection中的一个HashTable&lt;Integer, Call&gt;了，其中的Integer是用来标识Call，这样就可以将request和response对应上了。

**4、小结**

今天先休息一下了，吃坏东西了，悲催。
  > **参考资料：**
> 
> [智障大师的专栏](http://blog.csdn.net/historyasamirror/article/details/6159248)