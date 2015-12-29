title: 设计模式 - 适配器模式
tags:
  - 设计模式
id: 238
categories:
  - 技术分享
date: 2011-12-26 22:31:00
---

**GOF****定义：**

_“Convert the interface of a class into another interface clients expect. Adapter lets classed work together thar couldn’t otherwise because of incompatible interfaces.” _

_“将一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作。”_
 <!--more-->  

国外用的电压和国内的不一样，有110V的330V的，为了让水货电器更好的适配大陆地区，需要额外附加一个适配器。这个和我们所说的适配器模式差不多，都是在不变更对象的情况下，让原本不适合的对象适配起来。

如系统升级的时候，需要考虑到新系统的兼容性，就需要有一个适配器，接受低版本的请求；写多客户端或者多语言的项目的时候，也需要添加适配器，能够处理不同的请求；Android开发中为了让数据能够显示在界面上，也大量用到了适配器模式。

如做一个查询系统，有IBasicInfo以及IEducationInfo两个接口，分别获得用户的基本信息以及教育信息。    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.adapter;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> BasicInfo </span><span class="keyword">implements</span><span> IBasicInfo {&#160;&#160; </span></span>
4.  <span>&#160; </span>
5.  <span>&#160;&#160;&#160; </span><span class="comment">/** </span>&#160;</span>6.  <span><span class="comment">&#160;&#160;&#160;&#160; * 获得基本信息 </span>&#160;</span>7.  <span><span class="comment">&#160;&#160;&#160;&#160; */</span><span>&#160; </span></span>
8.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
9.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> Map&lt;String, String&gt; getBasicInfo() {&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Map&lt;String, String&gt; map = </span><span class="keyword">new</span><span> HashMap&lt;String, String&gt;();&#160;&#160; </span></span>
11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; map.put(</span><span class="string">&quot;Name&quot;</span><span>, </span><span class="string">&quot;Wikie&quot;</span><span>);&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; map.put(</span><span class="string">&quot;Gender&quot;</span><span>, </span><span class="string">&quot;male&quot;</span><span>);&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> map;&#160;&#160; </span></span>
14.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
15.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>        <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.adapter;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">import</span><span> java.util.HashMap;&#160;&#160; </span></span>
4.  <span></span><span class="keyword">import</span><span> java.util.Map;&#160;&#160; </span></span>
5.  <span>&#160; </span>
6.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> EducationInfo </span><span class="keyword">implements</span><span> IEducationInfo {&#160;&#160; </span></span>
7.  <span>&#160; </span>
8.  <span>&#160;&#160;&#160; </span><span class="comment">/** </span>&#160;</span>9.  <span><span class="comment">&#160;&#160;&#160;&#160; * 获得教育信息 </span>&#160;</span>10.  <span><span class="comment">&#160;&#160;&#160;&#160; */</span><span>&#160; </span></span>
11.  <span>&#160;&#160;&#160; </span><span class="annotation">@Override</span><span>&#160; </span></span>
12.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span> Map&lt;String, String&gt; getEducationInfo() {&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; Map&lt;String, String&gt; map = </span><span class="keyword">new</span><span> HashMap&lt;String, String&gt;();&#160;&#160; </span></span>
14.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; map.put(</span><span class="string">&quot;Master&quot;</span><span>, </span><span class="string">&quot;BUAA&quot;</span><span>);&#160;&#160; </span></span>
15.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; map.put(</span><span class="string">&quot;Major&quot;</span><span>, </span><span class="string">&quot;Software Engineering&quot;</span><span>);&#160;&#160; </span></span>
16.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">return</span><span> map;&#160;&#160; </span></span>
17.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
18.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

使用的时候就比较简单了，    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.adapter;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> Application {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">static</span><span>&#160;</span><span class="keyword">void</span><span> main(String[] args) {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; IBasicInfo basicInfo = </span><span class="keyword">new</span><span> BasicInfo();&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; IEducationInfo educationInfo = </span><span class="keyword">new</span><span> EducationInfo();&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.out.println(basicInfo.getBasicInfo());&#160;&#160; </span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.out.println(educationInfo.getEducationInfo());&#160;&#160; </span>
9.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
10.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

但是后面一拍脑袋，还有工作信息，家庭信息等等等，再继续创建接口与实现类？继续创建吧，但是如果我还想要获得所有信息呢？单独创建每个对象，再单独获得信息，这个就比较麻烦了，不过直接写一个适配器吧！    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.adapter;&#160;&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> Application {&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">static</span><span>&#160;</span><span class="keyword">void</span><span> main(String[] args) {&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; IBasicInfo basicInfo = </span><span class="keyword">new</span><span> BasicInfo();&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; IEducationInfo educationInfo = </span><span class="keyword">new</span><span> EducationInfo();&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.out.println(basicInfo.getBasicInfo());&#160;&#160; </span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.out.println(educationInfo.getEducationInfo());&#160;&#160; </span>
9.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
10.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

使用的时候：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <div class="dp-highlighter">             <div class="bar"></div>              

1.  <span><span class="keyword">package</span><span> com.wikieno.adapter;&#160;&#160; </span></span>2.  <span>&#160; </span>3.  <span></span><span class="keyword">public</span><span>&#160;</span><span class="keyword">class</span><span> Application {&#160;&#160; </span></span>4.  <span>&#160;&#160;&#160; </span><span class="keyword">public</span><span>&#160;</span><span class="keyword">static</span><span>&#160;</span><span class="keyword">void</span><span> main(String[] args) {&#160;&#160; </span></span>5.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; IInfoAdapter infoAdapter = </span><span class="keyword">new</span><span> InfoAdapter();&#160;&#160; </span></span>6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; System.out.println(infoAdapter.getInfo());&#160;&#160; </span>7.  <span>&#160;&#160;&#160; }&#160;&#160; </span>8.  <span>}&#160;&#160; </span>           </div>         </td>       </tr>     </tbody></table> 

最开始的时候我在想：如果设计之初就设计一个IInfo接口，那么什么麻烦都没有。但是设计总是会有纰漏的，适配器模式就是一个“补缺”的工具，用来解决接口不相容的问题。

适配器的优点：

**1、 ****关联不同的类**

只要你愿意，你就可以让任何两个没有一点关系的类一起运行，只要适配器写得好。

**2、 ****提高了复用度**

原有的代码还能在原来的系统中用，新增的功能也很好的使用到了旧代码，很好的提高了代码复用度。

**3、 ****非常灵活**

适配器可以任意添加删除，对系统没有任何影响。
