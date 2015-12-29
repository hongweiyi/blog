title: 设计模式 - 抽象工厂模式
tags:
  - 设计模式
id: 215
categories:
  - 技术分享
date: 2011-12-16 00:58:38
---

**GOF定义：**

_“Provide an interface for creating familiesof related or dependent objects withoutspecifying their concrete classes.”_

_“提供一个创建一系列相互依赖对象的接口，而无需指定它们具体的类。”_
 <!--more-->  

如果我们需要实例化很多对象，而且这些对象之间有一定的关系；

如果由于需求的变化，需要创建更多系列的对象。

这个时候就可以开始考虑使用抽象工厂模式了。这种模式提供一种“封装机制”来避免客户程序和这种“多系列具体对象创建工作”的紧耦合。

接上一篇工厂的数据库访问例子，原访问接口只有get与set两个方法，但是后面数据库更新了，发现还有一个自增长（incrby）的方法。这下该怎么做呢？修改原来的Command接口，这个不行，违背了开闭原则。既然不能修改原有的代码，那就只能扩展上层的代码了。

那么，在原有的代码基础上，新增Incrable接口：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.abstractfactory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">interface</span><span> Incrable {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="comment">/** </span>&#160;</span>5.  <span><span class="comment">&#160;&#160;&#160;&#160; * 自增长某个字段 </span>&#160;</span>6.  <span><span class="comment">&#160;&#160;&#160;&#160; * @param args 参数数组 </span>&#160;</span>7.  <span><span class="comment">&#160;&#160;&#160;&#160; * @param step 增长步长 </span>&#160;</span>8.  <span><span class="comment">&#160;&#160;&#160;&#160; * @return 增长后的长度 </span>&#160;</span>9.  <span><span class="comment">&#160;&#160;&#160;&#160; */</span><span>&#160; </span></span>
10.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">int</span><span> incrby(String[] args, </span><span class="keyword">int</span><span> step);&#160;&#160; </span></span>
11.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

且分别写两个实现类，HashIncrable与SimpleIncrable：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.abstractfactory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> HashIncrable </span><span class="keyword">implements</span><span> Incrable {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
5.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">int</span><span> incrby(String[] args, </span><span class="keyword">int</span><span> step) {&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; String key, field;&#160;&#160; </span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">int</span><span> result = -</span><span class="number">1</span><span>;&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (</span><span class="number">2</span><span> == args.length) {&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; key = args[</span><span class="number">0</span><span>];&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; field = args[</span><span class="number">1</span><span>];&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span> {&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">throw</span><span>&#160;</span><span class="keyword">new</span><span> IllegalArgumentException(</span><span class="string">&quot;参数输入不对&quot;</span><span>);&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 获得client对象，增长相应值 </span><span>&#160; </span></span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// sample: result = conn.incrby.....; </span><span>&#160; </span></span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> result;&#160;&#160; </span></span>
17.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
18.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>      <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.abstractfactory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> SimpleIncrable </span><span class="keyword">implements</span><span> Incrable {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
5.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">int</span><span> incrby(String[] args, </span><span class="keyword">int</span><span> step) {&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; String key;&#160;&#160; </span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">int</span><span> result = -</span><span class="number">1</span><span>;&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (</span><span class="number">1</span><span> == args.length) {&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; key = args[</span><span class="number">0</span><span>];&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span> {&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">throw</span><span>&#160;</span><span class="keyword">new</span><span> IllegalArgumentException(</span><span class="string">&quot;参数输入不对&quot;</span><span>);&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 获得client对象，增长相应值 </span><span>&#160; </span></span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// sample: result = conn.incrby.....; </span><span>&#160; </span></span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> result;&#160;&#160; </span></span>
16.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
17.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

新增一个AbstactFactory的抽象类：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.abstractfactory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="comment">/** </span>&#160;</span>4.  <span><span class="comment">*&#160; </span>&#160;</span>5.  <span><span class="comment">* 抽象工厂类，可生产相应的对象 </span>&#160;</span>6.  <span><span class="comment">*&#160; </span>&#160;</span>7.  <span><span class="comment">* @author 易鸿伟 Wikie </span>&#160;</span>8.  <span><span class="comment">* @date 2011-12-15 </span>&#160;</span>9.  <span><span class="comment">*/</span><span>&#160; </span></span>
10.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">abstract</span><span>&#160;</span><span class="keyword">class</span><span> AbstractFactory {&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">abstract</span><span> Command createCmd();&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">abstract</span><span> Incrable createIncrable();&#160;&#160; </span></span>
13.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

分别写两个类，实现AbstractFactory抽象类：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.abstractfactory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="comment">/** </span>&#160;</span>4.  <span><span class="comment">*&#160; </span>&#160;</span>5.  <span><span class="comment">* Hash工厂，可以创建操作Hash结构的对象 </span>&#160;</span>6.  <span><span class="comment">*&#160; </span>&#160;</span>7.  <span><span class="comment">* @author 易鸿伟 Wikie </span>&#160;</span>8.  <span><span class="comment">* @date 2011-12-15 </span>&#160;</span>9.  <span><span class="comment">*/</span><span>&#160; </span></span>
10.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> HashFactory </span><span class="keyword">extends</span><span> AbstractFactory {&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
12.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> Command createCmd() {&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span>&#160;</span><span class="keyword">new</span><span> HashCommand();&#160;&#160; </span></span>
14.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
15.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
16.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> Incrable createIncrable() {&#160;&#160; </span></span>
17.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span>&#160;</span><span class="keyword">new</span><span> HashIncrable();&#160;&#160; </span></span>
18.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
19.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.abstractfactory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="comment">/** </span>&#160;</span>4.  <span><span class="comment">*&#160; </span>&#160;</span>5.  <span><span class="comment">* Simple工厂，可以创建操作Simple结构的对象 </span>&#160;</span>6.  <span><span class="comment">*&#160; </span>&#160;</span>7.  <span><span class="comment">* @author 易鸿伟 Wikie </span>&#160;</span>8.  <span><span class="comment">* @date 2011-12-15 </span>&#160;</span>9.  <span><span class="comment">*/</span><span>&#160; </span></span>
10.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> SimpleFactory </span><span class="keyword">extends</span><span> AbstractFactory {&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
12.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> Command createCmd() {&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span>&#160;</span><span class="keyword">new</span><span> SimpleCommand();&#160;&#160; </span></span>
14.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
15.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
16.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> Incrable createIncrable() {&#160;&#160; </span></span>
17.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span>&#160;</span><span class="keyword">new</span><span> SimpleIncrable();&#160;&#160; </span></span>
18.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
19.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

最后就这样使用它：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.abstractfactory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> Application {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">static</span><span>&#160;</span><span class="keyword">void</span><span> main(String[] args) {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; AbstractFactory factory = </span><span class="keyword">new</span><span> HashFactory();&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Command cmd = factory.createCmd();&#160;&#160; </span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Incrable incrable = factory.createIncrable();&#160;&#160; </span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// cmd.get(args); </span><span>&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// cmd.set(args); </span><span>&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// incrable.incrby(args, step); </span><span>&#160; </span></span>
11.  <span>&#160; </span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; factory = </span><span class="keyword">new</span><span> SimpleFactory();&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; cmd = factory.createCmd();&#160;&#160; </span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; incrable = factory.createIncrable();&#160;&#160; </span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// cmd.get(args); </span><span>&#160; </span></span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// cmd.set(args); </span><span>&#160; </span></span>
17.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// incrable.incrby(args, step); </span><span>&#160; </span></span>
18.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
19.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

关于工厂模式与抽象工厂模式的比较，等到主要设计模式总结完后，再一起比较吧。
