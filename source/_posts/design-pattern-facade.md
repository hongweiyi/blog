title: 设计模式 - 门面模式
tags:
  - 设计模式
id: 235
categories:
  - 技术分享
date: 2011-12-26 20:29:00
---

**GOF****定义：**

_“Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.” _
 <!--more-->  

_“要求一个子系统的外部与其内部的通信必须通过一个统一的对象进行。门面模式提供了一个高层次的接口，使得子系统更易于使用。”_

门面模式给用户提供了一个使用子系统的接口（或者说界面），让用户只能通过这个接口来访问子系统。即门面对象是外界访问子系统内部的唯一通道。

每个应用程序其实都是一个门面，我们只能通过界面组件来访问系统内部资源，用起来非常简单的同时我们也不需要知道内部是怎么使用的（也不应该让用户深入内部）。

如用户管理子系统：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.facade;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> UserFacade {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">private</span><span> LoginModule loginModule = </span><span class="keyword">new</span><span> LoginModule();&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160; </span><span class="keyword">private</span><span> RegModule regModule = </span><span class="keyword">new</span><span> RegModule();&#160;&#160; </span></span>
6.  <span>&#160; </span>
7.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> reg(String name, String pwd, String info) {&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; regModule.reg(name, pwd, info);&#160;&#160; </span>
9.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
10.  <span>&#160; </span>
11.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">void</span><span> login(String name, String pwd) {&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; loginModule.login(name, pwd);&#160;&#160; </span>
13.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
14.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

在LoginModule以及RegModule中分别实现了login以及reg的逻辑。

门面模式的优缺点：

**1、 减少系统的相互依赖**

减少了用户或者其它子系统与子系统之间的依赖关系，让它们依赖的是门面，而不是子系统本身。

**2、 提供了灵活性**

减少依赖关系，灵活性自然就提高了。无论子系统内部如何变化，只要门面没有变，外部就没有影响。

**3、 提高安全性**

用户不知道内部实现的逻辑，也不能访问到内部实现，提高了安全性。

门面模式的缺点也显而易见，不符合开闭原则。如果还有功能需要添加的话，继承复用均没用，只能修改门面的代码。所以使用之前，需要多思考思考。