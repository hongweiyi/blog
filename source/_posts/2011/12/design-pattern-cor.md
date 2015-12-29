title: 设计模式 - 责任链模式
tags:
  - 设计模式
id: 229
categories:
  - 技术分享
date: 2011-12-22 10:51:28
---

**GOF****定义：**

_“Avoid coupling thesender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it” _

_“使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。”_
 <!--more-->  

如果你有多个类似的处理逻辑，但是事先不知道到底应该由哪一个处理，你可以考虑使用一下责任链模式。

用户将请求发到责任链的起始对象，如果该对象无法解决问题，则将请求发往链的下一个对象，依此迭代，直到有对象处理请求或者无人处理直接返回。不过在一个纯的责任链模式中，不会出现无人处理的情况，一个请求必须被某一个处理对象所接受。

想象一下腾讯的等级加速情况，普通用户以及VIP1-6（听说快出7了），每个等级都有自己的天数加速策略。我们可以模拟一下，下面这个是加速器抽象类：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.chainofresponsibility;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">abstract</span><span>&#160;</span><span class="keyword">class</span><span> Speeder {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">protected</span><span> Speeder next;&#160;&#160; </span></span>
5.  <span>&#160; </span>
6.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> setNext(Speeder next) {&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Speeder tmp = </span><span class="keyword">this</span><span>;&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">while</span><span> (</span><span class="keyword">null</span><span> != tmp.next) {&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; tmp = tmp.next;&#160;&#160; </span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; tmp.next = next;&#160;&#160; </span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; tmp = </span><span class="keyword">null</span><span>;&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
14.  <span>&#160; </span>
15.  <span>&#160;&#160;&#160; </span><span class="comment">/** </span>&#160;</span>16.  <span><span class="comment">&#160;&#160;&#160;&#160; * 判断是否还有下一个结点 </span>&#160;</span>17.  <span><span class="comment">&#160;&#160;&#160;&#160; *&#160; </span>&#160;</span>18.  <span><span class="comment">&#160;&#160;&#160;&#160; * @return </span>&#160;</span>19.  <span><span class="comment">&#160;&#160;&#160;&#160; */</span><span>&#160; </span></span>
20.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">boolean</span><span> hasNext() {&#160;&#160; </span></span>
21.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (</span><span class="keyword">null</span><span> != next) {&#160;&#160; </span></span>
22.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span>&#160;</span><span class="keyword">true</span><span>;&#160;&#160; </span></span>
23.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
24.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span>&#160;</span><span class="keyword">false</span><span>;&#160;&#160; </span></span>
25.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
26.  <span>&#160; </span>
27.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">abstract</span><span>&#160;</span><span class="keyword">void</span><span> speed(User user);&#160;&#160; </span></span>
28.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">abstract</span><span>&#160;</span><span class="keyword">boolean</span><span> canHandle(User user);&#160;&#160; </span></span>
29.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

普通用户加速器：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.chainofresponsibility;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> SimpleSpeeder </span><span class="keyword">extends</span><span> Speeder {&#160;&#160; </span></span>
4.  <span>&#160; </span>
5.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
6.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> speed(User user) {&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (canHandle(user)) {&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TODO </span><span>&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span> {&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (hasNext()) {&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; next.speed(user);&#160;&#160; </span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
14.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
15.  <span>&#160; </span>
16.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
17.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">boolean</span><span> canHandle(User user) {&#160;&#160; </span></span>
18.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (!user.isVIP) {&#160;&#160; </span></span>
19.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span>&#160;</span><span class="keyword">true</span><span>;&#160;&#160; </span></span>
20.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
21.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span>&#160;</span><span class="keyword">false</span><span>;&#160;&#160; </span></span>
22.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
23.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

VIP1加速器：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.chainofresponsibility;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> VIP1Speeder </span><span class="keyword">extends</span><span> Speeder {&#160;&#160; </span></span>
4.  <span>&#160; </span>
5.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">static</span><span>&#160;</span><span class="keyword">int</span><span> LEVEL = </span><span class="number">1</span><span>;&#160;&#160; </span></span>
6.  <span>&#160; </span>
7.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
8.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> speed(User user) {&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (canHandle(user)) {&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TODO </span><span>&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.out.println(</span><span class="number">1</span><span>+</span><span class="string">&quot;&quot;</span><span>);&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span> {&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (hasNext()) {&#160;&#160; </span></span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; next.speed(user);&#160;&#160; </span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
17.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
18.  <span>&#160; </span>
19.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
20.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">boolean</span><span> canHandle(User user) {&#160;&#160; </span></span>
21.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (LEVEL == user.levelVIP) {&#160;&#160; </span></span>
22.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span>&#160;</span><span class="keyword">true</span><span>;&#160;&#160; </span></span>
23.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
24.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span>&#160;</span><span class="keyword">false</span><span>;&#160;&#160; </span></span>
25.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
26.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

VIP2同上，就只修改了LEVEL常量。

使用的时候：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.chainofresponsibility;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> Application {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">static</span><span>&#160;</span><span class="keyword">void</span><span> main(String[] args) {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; User user = </span><span class="keyword">new</span><span> User();&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; user.isVIP = </span><span class="keyword">true</span><span>;&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; user.levelVIP = </span><span class="number">2</span><span>;&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Speeder speeder = </span><span class="keyword">new</span><span> SimpleSpeeder();&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; speeder.setNext(</span><span class="keyword">new</span><span> VIP1Speeder());&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; speeder.setNext(</span><span class="keyword">new</span><span> VIP2Speeder());&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; speeder.speed(user);&#160;&#160; </span>
13.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
14.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

这个只是用责任链模式简单模拟了一下QQ等级加速的情况，并不代表现实情况。

显然，VIP6加速就得往下传递6次，有点浪费资源，但是如果腾讯再推出一个VIP7，又有不同的speed策略怎么办呢？用责任链的话，只需要添加一个VIP7Speeder类，在speeder链中加入这个结点即可。在原来的基础之上，不需要修改，只需要在应用中添加一行代码，符合开闭原则。同理，在设计之初，也可以加入删除的接口。
