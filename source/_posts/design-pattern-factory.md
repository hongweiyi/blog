title: 设计模式 - 工厂模式
tags:
  - 设计模式
id: 213
categories:
  - 技术分享
date: 2011-12-16 00:47:04
---

**GOF定义：**

“_Define an interface for creating an object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses._”

“_定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method使一个类的实例化延迟到其子类。_”
 <!--more-->如果我们经常需要实例化很多对象，如：A a = new A(); B b = new B();     

如果在开发过程中，我们发现，类A、B的方法是那么的一致，但是具体实现却大不太一样。

这个时候就可以开始考虑使用工厂模式了。工厂模式提供一个封装机制来隔离出a、b对象的具体实现，提炼出一致的接口，从而保持了系统中其它依赖这些接口的对象不会随着需求的改变而改变，即解耦。用户无需知道a、b对象具体是怎么创建的，只管用即可。

程序猿对数据库一定不会陌生，近几年流行一种新的数据库——NoSQL，NoSQL中又流行Key/Value内存数据库。故名思议，数据库中存放的都是一些键值对。

一般来说，操作键值对就两个方法：get(key)与set(key,value)。这两个方法应对简单结构的数据库足够了，但是有丰富数据结构的就不行了，如redis。虽然说具体实现不太一样，但是接口名基本上是一致的，还是get与set，只是传入的参数个数不同。那么就可以抽出有这样两个方法的接口：get(String[])与set(String[])。
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.factory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">interface</span><span> Command {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="comment">/** </span>&#160;</span>5.  <span><span class="comment">&#160;&#160;&#160;&#160; * 根据参数取得值 </span>&#160;</span>6.  <span><span class="comment">&#160;&#160;&#160;&#160; * @param args 参数数组 </span>&#160;</span>7.  <span><span class="comment">&#160;&#160;&#160;&#160; * @return 值 </span>&#160;</span>8.  <span><span class="comment">&#160;&#160;&#160;&#160; */</span><span>&#160; </span></span>
9.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> String get(String[] args);&#160;&#160; </span></span>
10.  <span>&#160; </span>
11.  <span>&#160;&#160;&#160; </span><span class="comment">/** </span>&#160;</span>12.  <span><span class="comment">&#160;&#160;&#160;&#160; * 根据参数设置值 </span>&#160;</span>13.  <span><span class="comment">&#160;&#160;&#160;&#160; * @param args 参数数组 </span>&#160;</span>14.  <span><span class="comment">&#160;&#160;&#160;&#160; * @return 影响条数，-1为失败 </span>&#160;</span>15.  <span><span class="comment">&#160;&#160;&#160;&#160; */</span><span>&#160; </span></span>
16.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">int</span><span> set(String[] args);&#160;&#160; </span></span>
17.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

同时，有一些类实现了这个接口，本文简单的使用了HashCommand与SimpleCommand：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.factory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="comment">/** </span>&#160;</span>4.  <span><span class="comment">*&#160; </span>&#160;</span>5.  <span><span class="comment">* 对Redis的Hash结构进行操作的类 </span>&#160;</span>6.  <span><span class="comment">*&#160; </span>&#160;</span>7.  <span><span class="comment">* @author 易鸿伟 Wikie </span>&#160;</span>8.  <span><span class="comment">* @date 2011-12-15 </span>&#160;</span>9.  <span><span class="comment">*/</span><span>&#160; </span></span>
10.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> HashCommand </span><span class="keyword">implements</span><span> Command {&#160;&#160; </span></span>
11.  <span>&#160; </span>
12.  <span>&#160;&#160;&#160; </span><span class="keyword">private</span><span>&#160;</span><span class="keyword">static</span><span> String KEY = </span><span class="string">&quot;default_key&quot;</span><span>;&#160;&#160; </span></span>
13.  <span>&#160; </span>
14.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
15.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> String get(String[] args) {&#160;&#160; </span></span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; String key, field, value = </span><span class="keyword">null</span><span>;&#160;&#160; </span></span>
17.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (</span><span class="number">2</span><span> == args.length) {&#160;&#160; </span></span>
18.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; key = args[</span><span class="number">0</span><span>];&#160;&#160; </span></span>
19.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; field = args[</span><span class="number">1</span><span>];&#160;&#160; </span></span>
20.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span>&#160;</span><span class="keyword">if</span><span> (</span><span class="number">1</span><span> == args.length) {&#160;&#160; </span></span>
21.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; key = KEY;&#160;&#160; </span>
22.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; field = args[</span><span class="number">0</span><span>];&#160;&#160; </span></span>
23.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span> {&#160;&#160; </span></span>
24.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">throw</span><span>&#160;</span><span class="keyword">new</span><span> IllegalArgumentException(</span><span class="string">&quot;参数输入不对&quot;</span><span>);&#160;&#160; </span></span>
25.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
26.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 获得client对象，获得数据 </span><span>&#160; </span></span>
27.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// sample: value = conn.get.....; </span><span>&#160; </span></span>
28.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> value;&#160;&#160; </span></span>
29.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
30.  <span>&#160; </span>
31.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
32.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">int</span><span> set(String[] args) {&#160;&#160; </span></span>
33.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; String key, field, value;&#160;&#160; </span>
34.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">int</span><span> result = -</span><span class="number">1</span><span>;&#160;&#160; </span></span>
35.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (</span><span class="number">3</span><span> == args.length) {&#160;&#160; </span></span>
36.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; key = args[</span><span class="number">0</span><span>];&#160;&#160; </span></span>
37.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; field = args[</span><span class="number">1</span><span>];&#160;&#160; </span></span>
38.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; value = args[</span><span class="number">2</span><span>];&#160;&#160; </span></span>
39.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span>&#160;</span><span class="keyword">if</span><span> (</span><span class="number">2</span><span> == args.length) {&#160;&#160; </span></span>
40.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; key = KEY;&#160;&#160; </span>
41.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; field = args[</span><span class="number">0</span><span>];&#160;&#160; </span></span>
42.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; value = args[</span><span class="number">1</span><span>];&#160;&#160; </span></span>
43.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span> {&#160;&#160; </span></span>
44.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">throw</span><span>&#160;</span><span class="keyword">new</span><span> IllegalArgumentException(</span><span class="string">&quot;参数输入不对&quot;</span><span>);&#160;&#160; </span></span>
45.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
46.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 获得client对象，设置数据 </span><span>&#160; </span></span>
47.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// sample: result = conn.set.....; </span><span>&#160; </span></span>
48.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> result;&#160;&#160; </span></span>
49.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
50.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.factory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="comment">/** </span>&#160;</span>4.  <span><span class="comment">*&#160; </span>&#160;</span>5.  <span><span class="comment">* 对Redis的普通结构（Key/Value）进行操作的类 </span>&#160;</span>6.  <span><span class="comment">*&#160; </span>&#160;</span>7.  <span><span class="comment">*&#160; </span>&#160;</span>8.  <span><span class="comment">* @author 易鸿伟 Wikie </span>&#160;</span>9.  <span><span class="comment">* @date 2011-12-15 </span>&#160;</span>10.  <span><span class="comment">*/</span><span>&#160; </span></span>
11.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> SimpleCommand </span><span class="keyword">implements</span><span> Command {&#160;&#160; </span></span>
12.  <span>&#160; </span>
13.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> String get(String[] args) {&#160;&#160; </span></span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; String key, value = </span><span class="keyword">null</span><span>;&#160;&#160; </span></span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (</span><span class="number">1</span><span> == args.length) {&#160;&#160; </span></span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; key = args[</span><span class="number">0</span><span>];&#160;&#160; </span></span>
17.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span> {&#160;&#160; </span></span>
18.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">throw</span><span>&#160;</span><span class="keyword">new</span><span> IllegalArgumentException(</span><span class="string">&quot;参数输入不对&quot;</span><span>);&#160;&#160; </span></span>
19.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
20.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 获得client对象，获得数据 </span><span>&#160; </span></span>
21.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// sample: value = conn.get.....; </span><span>&#160; </span></span>
22.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> value;&#160;&#160; </span></span>
23.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
24.  <span>&#160; </span>
25.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">int</span><span> set(String[] args) {&#160;&#160; </span></span>
26.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; String key, value;&#160;&#160; </span>
27.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">int</span><span> result = -</span><span class="number">1</span><span>;&#160;&#160; </span></span>
28.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (</span><span class="number">2</span><span> == args.length) {&#160;&#160; </span></span>
29.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; key = args[</span><span class="number">0</span><span>];&#160;&#160; </span></span>
30.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; value = args[</span><span class="number">1</span><span>];&#160;&#160; </span></span>
31.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span> {&#160;&#160; </span></span>
32.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">throw</span><span>&#160;</span><span class="keyword">new</span><span> IllegalArgumentException(</span><span class="string">&quot;参数输入不对&quot;</span><span>);&#160;&#160; </span></span>
33.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
34.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// 获得client对象，设置数据 </span><span>&#160; </span></span>
35.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// sample: result = conn.set.....; </span><span>&#160; </span></span>
36.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> result;&#160;&#160; </span></span>
37.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
38.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

工厂类实现就比较简单了，只需要有一个静态create方法即可。
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.factory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> CommandFactory {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">static</span><span> Command create(String className) {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Command cmd = </span><span class="keyword">null</span><span>;&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">try</span><span> {&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; cmd = (Command) Class.forName(className).newInstance();&#160;&#160; </span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">catch</span><span> (InstantiationException e) {&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TOOD 实例化出错，即传入的不是具体类名 </span><span>&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">catch</span><span> (IllegalAccessException e) {&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TOOD 定义的具体类有问题 </span><span>&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">catch</span><span> (ClassNotFoundException e) {&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TOOD 具体类无法找到 </span><span>&#160; </span></span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> cmd;&#160;&#160; </span></span>
16.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
17.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

在最后的程序中，用户只需要定义一个接口，再将需要实例化的类名传给工厂即可。其中类名常量存在了一个Utils类中，为了减少篇幅就不添加进来了。
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="bar"></div>          <div class="dp-highlighter">           

1.  <span><span class="keyword">package</span><span> com.wikieno.factory;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> Application {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">static</span><span>&#160;</span><span class="keyword">void</span><span> main(String[] args) {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; String[] hGetArgs = {</span><span class="string">&quot;key&quot;</span><span>, </span><span class="string">&quot;field&quot;</span><span>};&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; String[] hSetArgs = {</span><span class="string">&quot;key&quot;</span><span>, </span><span class="string">&quot;field&quot;</span><span>, </span><span class="string">&quot;value&quot;</span><span>};&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; String[] sGetArgs = {</span><span class="string">&quot;key&quot;</span><span>};&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; String[] sSetArgs = {</span><span class="string">&quot;key&quot;</span><span>, </span><span class="string">&quot;value&quot;</span><span>};&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Command cmd = CommandFactory.create(Utils.CLASS_HASH);&#160;&#160; </span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; cmd.set(hSetArgs);&#160;&#160; </span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; cmd.get(hGetArgs);&#160;&#160; </span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; cmd = CommandFactory.create(Utils.CLASS_SIMPLE);&#160;&#160; </span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; cmd.set(sSetArgs);&#160;&#160; </span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; cmd.get(sGetArgs);&#160;&#160; </span>
17.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
18.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>