title: 设计模式 - 装饰模式
tags:
  - 设计模式
id: 234
categories:
  - 技术分享
date: 2011-12-25 23:41:03
---

**GOF****定义：**

_“Attach additional responsibilities to an object dynamically keeping the same interface. Decorators provide a flexible alternative to subclassing for extending functionality.” _

_“动态地给一个对象添加一些额外的职责，就增加功能来说，装饰模式相比生成子类更为灵活。”_
 <!--more-->  

如果你想给某个类添加一个功能，但是觉得没有到继承的地步；如果直接修改新增方法，那么新增方法对子类的影响大不大，这个也不太好说。最好的方法就是通过新增一个Decorator类来修饰该类。

设计一个Mp3播放器：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.decorator;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> MP3PlayerComponent </span><span class="keyword">implements</span><span> Component {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> operate() {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TODO </span><span>&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.out.println(</span><span class="string">&quot;Play a mp3 now!&quot;</span><span>);&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
8.  <span>}&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

使用一段时间后，发现还想添加一些功能，比如说播放歌词，让修改吧，不符合开闭原则。继承吧，MP3PlayerWithLrcComponent，怎么看怎么怪。直接用一个修饰器是最方便的：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.decorator;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">abstract</span><span>&#160;</span><span class="keyword">class</span><span> Decorator </span><span class="keyword">implements</span><span> Component {&#160;&#160; </span></span>
4.  <span>&#160; </span>
5.  <span>&#160;&#160;&#160; </span><span class="keyword">private</span><span> Component component = </span><span class="keyword">null</span><span>;&#160;&#160; </span></span>
6.  <span>&#160; </span>
7.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> Decorator(Component component) {&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">this</span><span>.component = component;&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
10.  <span>&#160; </span>
11.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
12.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> operate() {&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">this</span><span>.component.operate();&#160;&#160; </span></span>
14.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
15.  <span>}&#160; </span>           </div>         </td>       </tr>        <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.decorator;&#160;&#160; </span></span>2.  <span>&#160; </span>3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> MP3Decorator </span><span class="keyword">extends</span><span> Decorator {&#160;&#160; </span></span>4.  <span>&#160; </span>5.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> MP3Decorator(Component component) {&#160;&#160; </span></span>6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">super</span><span>(component);&#160;&#160; </span></span>7.  <span>&#160;&#160;&#160; }&#160;&#160; </span>8.  <span>&#160; </span>9.  <span>&#160;&#160;&#160; </span><span class="comment">/** </span>&#160;</span>10.  <span><span class="comment">&#160;&#160;&#160;&#160; * 自己的装饰方法 显示歌词 </span>&#160;</span>11.  <span><span class="comment">&#160;&#160;&#160;&#160; */</span><span>&#160; </span></span>12.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> showLrc() {&#160;&#160; </span></span>13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TODO </span><span>&#160; </span></span>14.  <span>&#160;&#160;&#160; }&#160;&#160; </span>15.  <span>&#160; </span>16.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>17.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> operate() {&#160;&#160; </span></span>18.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">super</span><span>.operate();&#160;&#160; </span></span>19.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">this</span><span>.showLrc();&#160;&#160; </span></span>20.  <span>&#160;&#160;&#160; }&#160;&#160; </span>21.  <span>}&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

优点：

**1、 ****装饰器与被装饰类解耦**

装饰器在扩展类的功能的时候，与被装饰类完全解耦，独立运行。

**2、 ****符合合成复用原则**