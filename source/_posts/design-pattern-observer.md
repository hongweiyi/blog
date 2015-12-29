title: 设计模式 - 观察者模式
tags:
  - 设计模式
id: 225
categories:
  - 技术分享
date: 2011-12-18 21:17:41
---

**GOF****定义：**

_“Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically” _

_“定义对象间的一种一对多的依赖关系，当一个对象的状态发生变化时，所有依赖于它的对象都得到通知并被自动更新。”_
 <!--more-->  

写过桌面的程序的人一定不会陌生Listener这个东东，添加一个组件就得添加一个相应的XXXListener，这也是观察者模式一个比较常见的应用领域了。

先看监听器（观察者）的实现，实现中只有事件发生时调用的接口方法声明：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar">&#160;</div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.observer.listener;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="comment">/** </span>&#160;</span>4.  <span><span class="comment">*&#160; </span>&#160;</span>5.  <span><span class="comment">* 监听器（观察者）接口，具体实现由使用者实现 </span>&#160;</span>6.  <span><span class="comment">*&#160; </span>&#160;</span>7.  <span><span class="comment">* @author Wikie </span>&#160;</span>8.  <span><span class="comment">*/</span><span>&#160; </span></span>
9.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">abstract</span><span>&#160;</span><span class="keyword">class</span><span> SendListener{&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">abstract</span><span>&#160;</span><span class="keyword">void</span><span> onMsgSended(String msg);&#160;&#160; </span></span>
11.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

应用组件，即被观察者：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.observer.listener;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> Sender {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="comment">// 监听器（观察者） </span><span>&#160; </span></span>
5.  <span>&#160;&#160;&#160; SendListener sendListener;&#160;&#160; </span>
6.  <span>&#160; </span>
7.  <span>&#160;&#160;&#160; </span><span class="comment">/** </span>&#160;</span>8.  <span><span class="comment">&#160;&#160;&#160;&#160; * 添加（注册）一个监听器 </span>&#160;</span>9.  <span><span class="comment">&#160;&#160;&#160;&#160; * @param l 监听器 </span>&#160;</span>10.  <span><span class="comment">&#160;&#160;&#160;&#160; */</span><span>&#160; </span></span>
11.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> addSendListener(SendListener l) {&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (l == </span><span class="keyword">null</span><span>) {&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">throw</span><span>&#160;</span><span class="keyword">new</span><span> NullPointerException();&#160;&#160; </span></span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">this</span><span>.sendListener = l;&#160;&#160; </span></span>
16.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
17.  <span>&#160; </span>
18.  <span>&#160;&#160;&#160; </span><span class="comment">/** </span>&#160;</span>19.  <span><span class="comment">&#160;&#160;&#160;&#160; * 发送消息 </span>&#160;</span>20.  <span><span class="comment">&#160;&#160;&#160;&#160; * 同时通知监听器 </span>&#160;</span>21.  <span><span class="comment">&#160;&#160;&#160;&#160; * @param msg </span>&#160;</span>22.  <span><span class="comment">&#160;&#160;&#160;&#160; */</span><span>&#160; </span></span>
23.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> sendMsg(String msg) {&#160;&#160; </span></span>
24.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; doSth();&#160;&#160; </span>
25.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; sendListener.onMsgSended(msg);&#160;&#160; </span>
26.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
27.  <span>&#160; </span>
28.  <span>&#160;&#160;&#160; </span><span class="keyword">private</span><span>&#160;</span><span class="keyword">void</span><span> doSth() {&#160;&#160; </span></span>
29.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TODO Auto-generated method stubO </span><span>&#160; </span></span>
30.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
31.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

最后应用程序使用时，只需要在组件中注册一个监听器即可：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.observer.listener;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> Application {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">static</span><span>&#160;</span><span class="keyword">void</span><span> main(String[] args) {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sender sender = </span><span class="keyword">new</span><span> Sender();&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; sender.addSendListener(</span><span class="keyword">new</span><span> SendListener() {&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> onMsgSended(String msg) {&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.out.println(</span><span class="string">&quot;Msg is: &quot;</span><span> + msg);&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; });&#160;&#160; </span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; sender.sendMsg(</span><span class="string">&quot;Hello World&quot;</span><span>);&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
14.  <span>}&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

观察者模式也不仅仅局限于界面监听器，系统实时推送更新消息也可以考虑使用该模式。

需要注意的是，java中已经有现成的观察者模式了，即使用Observer接口与Observable类。不过出于线程安全的考虑，Sun JDK的实现可能会出现两个不太好的情况：1、新加入的观察者会丢失消息；2、新删除的观察者会收到之前的消息。对这个要求较高的建议重写吧，一般来说，我们还是可以使用它的。

他们俩内部实现和上面的代码差不多，Observer接口中只有一个update方法声明，而Observable中有一个存放Observer的Vector对象，可以给一个Observable对象注册多个Observer，同时发生事件后调用notifyObservers(obj)即可通知所有的Observer。

将上面代码改一下，就成了这个样子：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.observer;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">import</span><span> java.util.Observable;&#160;&#160; </span></span>
4.  <span></span><span class="keyword">import</span><span> java.util.Observer;&#160;&#160; </span></span>
5.  <span>&#160; </span>
6.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> SendObserver </span><span class="keyword">implements</span><span> Observer {&#160;&#160; </span></span>
7.  <span>&#160; </span>
8.  <span>&#160;&#160;&#160; </span><span class="comment">/** </span>&#160;</span>9.  <span><span class="comment">&#160;&#160;&#160;&#160; * 事件处理方法 </span>&#160;</span>10.  <span><span class="comment">&#160;&#160;&#160;&#160; * @param o: 发生事件的被观察者 </span>&#160;</span>11.  <span><span class="comment">&#160;&#160;&#160;&#160; * @param arg: 参数 </span>&#160;</span>12.  <span><span class="comment">&#160;&#160;&#160;&#160; */</span><span>&#160; </span></span>
13.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
14.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> update(Observable o, Object arg) {&#160;&#160; </span></span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (arg </span><span class="keyword">instanceof</span><span> String) {&#160;&#160; </span></span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.out.println(</span><span class="string">&quot;Msg is: &quot;</span><span> + arg);&#160;&#160; </span></span>
17.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
18.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
19.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.observer;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">import</span><span> java.util.Observable;&#160;&#160; </span></span>
4.  <span>&#160; </span>
5.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> Sender </span><span class="keyword">extends</span><span> Observable {&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> sendMsg(String msg) {&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; setChanged();&#160;&#160; </span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; notifyObservers(msg);&#160;&#160; </span>
9.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
10.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.observer;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> Application {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">static</span><span>&#160;</span><span class="keyword">void</span><span> main(String[] args) {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Sender sender = </span><span class="keyword">new</span><span> Sender();&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; sender.addObserver(</span><span class="keyword">new</span><span> SendObserver());&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; sender.sendMsg(</span><span class="string">&quot;Hello world&quot;</span><span>);&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
9.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>
