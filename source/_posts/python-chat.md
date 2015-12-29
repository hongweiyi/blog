title: 人生苦短，我用Python
tags:
  - Python
id: 243
categories:
  - 技术分享
date: 2012-01-20 16:40:51
---

用脚本语言的原因就是因为其语法简单，使用方便，但是好像一直没有把握其中的精华。虽然仔细读过网上转载过N次的&lt;[Python少打字小技巧](http://www.joynb.net/blog/archives/120)&gt;，但还是无法很好的运用。
 <!--more-->  

今天逛论坛的时候，看到了一个网友发了一个[ip地址排序](http://bbs.chinaunix.net/thread-823862-1-1.html)的帖子，代码行数60行，核心代码30多行。没有仔细读，看看评论的时候，看到了另一个解法，总代码6行，核心代码4行。用了列表解析、匿名函数等特性，还有内置函数，代码如下：
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span>iplist=['192.168.1.33','10.5.1.3','10.5.2.4','202.98.96.68','13.12.1.1']&#160;&#160; </span></span>
2.  <span></span><span class="keyword">def</span><span> ip2int(s):&#160;&#160; </span></span>
3.  <span>&#160;&#160;&#160; l = [</span><span class="builtins">int</span><span>(i) </span><span class="keyword">for</span><span> i </span><span class="keyword">in</span><span> s.split('.')]&#160;&#160; </span></span>
4.  <span>&#160;&#160;&#160; </span><span class="keyword">return</span><span> (l[0] &lt;&lt; 24) | (l[1] &lt;&lt; 16) | (l[2] &lt;&lt; 8 ) | l[3]&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160;&#160;&#160;&#160; </span>
6.  <span>iplist.sort(</span><span class="keyword">lambda</span><span> x, y: </span><span class="builtins">cmp</span><span>(ip2int(x), ip2int(y)))&#160;&#160; </span></span>
7.  <span></span><span class="keyword">print</span><span> iplist&#160; </span></span> </div>  

按照&lt;[Python少打字小技巧](http://www.joynb.net/blog/archives/120)&gt;里的例子，这个代码核心部分可以缩减到2行（其实可以只有1行），不过，可读性就不敢恭维咯。
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span>iplist=['192.168.1.33','10.5.1.3','10.5.2.4','202.98.96.68','13.12.1.1']&#160;&#160; </span></span>
2.  <span>p = </span><span class="keyword">lambda</span><span> ip: </span><span class="builtins">sum</span><span>( [ </span><span class="builtins">int</span><span>(k)*v </span><span class="keyword">for</span><span> k, v </span><span class="keyword">in</span><span>&#160;</span><span class="builtins">zip</span><span>(ip.split('.'), [1&lt;&lt;24, 65536, 256, 1])]);&#160;&#160; </span></span>
3.  <span>iplist.sort(</span><span class="keyword">lambda</span><span> x, y: </span><span class="builtins">cmp</span><span>(p(x), p(y)))&#160;&#160; </span></span>
4.  <span></span><span class="keyword">print</span><span> iplist&#160; </span></span> </div>  

既然说到短小精悍，就写写Python的一些特性和函数吧：lambda、map、reduce、filter

**lambda**

匿名函数，和c/c++中的宏定义类似，格式为：lambda [参数] : [表达式]，如：lambda x : x+2，调用的时候可以用一个变量接收这个函数，并传入参数。
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span>lam = </span><span class="keyword">lambda</span><span> x : x+2&#160;&#160; </span></span>
2.  <span></span><span class="keyword">print</span><span> lam(2)&#160;&#160; </span></span>
3.  <span></span><span class="keyword">print</span><span> (</span><span class="keyword">lambda</span><span> x : x+2)(2)&#160; </span></span> </div>  

有些人觉得lambda也就这点功能，但笔者认为，这才是脚本语言，点题：人生苦短，我用Python！这位[**博主**](http://www.cnblogs.com/coderzh/archive/2010/04/30/python-cookbook-lambda.html)写得不错，可以参考参考。

**map，reduce，filter**

其实lambda很多地方都是用在上面这三个函数上面，map(func,seq)、reduce(func,seq,initial=None)、filter(bool_func,seq)。

map：对seq中的每一个对象都执行func函数，并返回一个列表；

reduce：对seq中的每两个对象依次执行func函数，并返回一个值，如1+2+3+4；

filter：过滤掉seq中不符合bool_func的对象。

虽然这三个函数中func都可以使用def来定义，但是如果程序需要的逻辑比较简单的话，用lambda就足够了。
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span>ls = [1,2,3,4,5]&#160;&#160; </span></span>
2.  <span></span><span class="comment"># every item in ls plus 2 </span><span>&#160; </span></span>
3.  <span></span><span class="keyword">print</span><span>&#160;</span><span class="builtins">map</span><span>(</span><span class="keyword">lambda</span><span> x : x+2, ls)&#160;&#160; </span></span>
4.  <span></span><span class="comment"># sum(ls) </span><span>&#160; </span></span>
5.  <span></span><span class="keyword">print</span><span>&#160;</span><span class="builtins">reduce</span><span>(</span><span class="keyword">lambda</span><span> x, y : x+y, ls)&#160;&#160; </span></span>
6.  <span></span><span class="comment"># filter the item which less than 2 </span><span>&#160; </span></span>
7.  <span></span><span class="keyword">print</span><span>&#160;</span><span class="builtins">filter</span><span>(</span><span class="keyword">lambda</span><span> x : x &gt; 2 </span><span class="keyword">and</span><span>&#160;</span><span class="builtins">True</span><span>&#160;</span><span class="keyword">or</span><span>&#160;</span><span class="builtins">False</span><span>, ls)&#160;&#160; </span></span> </div>