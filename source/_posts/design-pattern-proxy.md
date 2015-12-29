title: 设计模式 - 代理模式
tags:
  - 设计模式
id: 233
categories:
  - 技术分享
date: 2011-12-25 23:37:24
---

**GOF****定义：**

_“Provide a surrogate or placeholder for another object to control access to it.” _

_“为其它对象提供一种代理以控制对这个对象的访问。”_
 <!--more-->  

为什么需要使用代理模式：

1、 如果不同用户对同一对象有不同的访问权利；

2、 如果某个用户不能直接操作某个对象，但是又必须和那个对象有所依赖。

有不同的访问权利，即定义中的控制对象访问，我们很容易就可以想到权限控制，比如说下载文件，交了钱的人才能下载，没交的没门。被下载文件有如下操作：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.proxy;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">interface</span><span> FileOps {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">void</span><span> download();&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160; </span><span class="keyword">void</span><span> del();&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160; </span><span class="keyword">void</span><span> modify();&#160;&#160; </span></span>
7.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>  

实际文件操作类如下：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.proxy;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> RealFileOps </span><span class="keyword">implements</span><span> FileOps {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> download() {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TODO </span><span>&#160; </span></span>
6.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
7.  <span>&#160; </span>
8.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> del() {&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TODO </span><span>&#160; </span></span>
10.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
11.  <span>&#160; </span>
12.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> modify() {&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TODO </span><span>&#160; </span></span>
14.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
15.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

文件操作代理类如下：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.proxy;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> FileProxy </span><span class="keyword">implements</span><span> FileOps {&#160;&#160; </span></span>
4.  <span>&#160; </span>
5.  <span>&#160;&#160;&#160; FileOps file;&#160;&#160; </span>
6.  <span>&#160;&#160;&#160; </span><span class="keyword">int</span><span> permission;&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">static</span><span>&#160;</span><span class="keyword">final</span><span>&#160;</span><span class="keyword">int</span><span> PERMISSION_USER = </span><span class="number">100</span><span>;&#160;&#160; </span></span>
8.  <span>&#160; </span>
9.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> FileProxy(</span><span class="keyword">int</span><span> permission, FileOps file) {&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// FIXME </span><span>&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">this</span><span>.file = file;&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">this</span><span>.permission = permission;&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
14.  <span>&#160; </span>
15.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
16.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> download() {&#160;&#160; </span></span>
17.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> (PERMISSION_USER == permission) {&#160;&#160; </span></span>
18.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">this</span><span>.file.download();&#160;&#160; </span></span>
19.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; } </span><span class="keyword">else</span><span> {&#160;&#160; </span></span>
20.  <span></span><span class="comment">// TODO </span><span>&#160; </span></span>
21.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160; </span>
22.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
23.  <span>&#160; </span>
24.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
25.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> del() {&#160;&#160; </span></span>
26.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TODO </span><span>&#160; </span></span>
27.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
28.  <span>&#160; </span>
29.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
30.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> modify() {&#160;&#160; </span></span>
31.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="comment">// TODO </span><span>&#160; </span></span>
32.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
33.  <span>}&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

优点：

1、 职责清晰

最大的优点就是职责清晰，真正的角色只需要实现实际的业务逻辑（本文的下载逻辑），而不用关心其它的非本职责的事务（权限管理）。

2、 高可扩展性

显而易见，在本文中，实际下载文件类实现了接口，需要添加新的实现时，只要将需要的新实例对象传入代理中即可。