title: 设计模式 - 单例模式
tags:
  - 设计模式
id: 224
categories:
  - 技术分享
date: 2011-12-17 22:52:21
---

**GOF****定义：**

_“Prototype Pattern Ensure a class only has one instance, and provide a global point of access to it.” _

_“保证一个类仅有一个实例，并提供一个访问它的全局访问点。”_
 <!--more-->  

如定义所说，单例模式确保了在同一时间内内存中又有类的一个实例，并提供了全局的访问方法。

节省内存是单例普遍认为的优点，那还有其它优点吗？当然还有，如果应用中存在多个（过多个）实例可能会引发程序（逻辑）错误时，也可以考虑用单例模式。比如说单一实例为池化对象，或唯一标识生成器等。

&#160;
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">package</span><span> com.wikieno.singleton;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> Singleton {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">private</span><span>&#160;</span><span class="keyword">static</span><span>&#160;</span><span class="keyword">final</span><span> Singleton mySingletonObj = </span><span class="keyword">new</span><span> Singleton();&#160;&#160; </span></span>
5.  <span>&#160; </span>
6.  <span>&#160;&#160;&#160; </span><span class="comment">// 不能直接创建实例 </span><span>&#160; </span></span>
7.  <span>&#160;&#160;&#160; </span><span class="keyword">private</span><span> Singleton() {&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
9.  <span>&#160; </span>
10.  <span>&#160;&#160;&#160; </span><span class="comment">// 直接返回实例 </span><span>&#160; </span></span>
11.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">synchronized</span><span>&#160;</span><span class="keyword">static</span><span> Singleton getInstance() {&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> mySingletonObj;&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
14.  <span>}&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>