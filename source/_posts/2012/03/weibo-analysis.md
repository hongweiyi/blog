title: '@小e_鸿伟的微博粗分析'
tags:
  - 微博
  - 数据分析
id: 492
categories:
  - 生活分享
date: 2012-03-26 23:20:04
---

**1****、杂侃**

最近状态着实不太好，也有点闲。大部分时间都在看书，coding也不太多，琢磨着写一两个小程序，也没找到好玩的。昨晚无聊刷微博的时候，灵机一动，干脆分析分析自己的微博吧。所以也就有了这篇文章。

<!--more-->

**2****、数据准备**

要分析，就得有数据，数据也还好弄，到了微博的[开放平台](http://open.weibo.com/)，下了一个Java SDK，开始拉数据……

微博的sdk比人人的易用一些，先登录获得access_token，再创建一个Weibo对象，将请求的json地址填进去就可以了。 
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span>User user = um.showUserById(uid);&#160;&#160; </span></span>2.  <span></span><span class="comment">// get the count of your weibos </span><span>&#160; </span></span>3.  <span></span><span class="comment">// prepared for iteration </span><span>&#160; </span></span>4.  <span></span><span class="keyword">int</span><span> count = user.getStatusesCount();&#160;&#160; </span></span>5.  <span></span><span class="keyword">int</span><span> page_count = </span><span class="number">30</span><span>;&#160;&#160; </span></span>6.  <span>String json = </span><span class="string">&quot;https://api.weibo.com/2/statuses/user_timeline.json?count=&quot;</span><span>&#160; </span></span>7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; + page_count + </span><span class="string">&quot;&amp;page=&quot;</span><span>;&#160;&#160; </span></span>8.  <span></span><span class="keyword">int</span><span> page = </span><span class="number">1</span><span>;&#160;&#160; </span></span>9.  <span>FileWriter writer;&#160;&#160; </span>10.  <span>Response response = </span><span class="keyword">null</span><span>;&#160;&#160; </span></span>11.  <span></span><span class="keyword">do</span><span> {&#160;&#160; </span></span>12.  <span>&#160;&#160;&#160; writer = </span><span class="keyword">new</span><span> FileWriter(</span><span class="keyword">new</span><span> File(</span><span class="string">&quot;txt&quot;</span><span>+page+</span><span class="string">&quot;.txt&quot;</span><span>), </span><span class="keyword">true</span><span>);&#160;&#160; </span></span>13.  <span>&#160;&#160;&#160; response = weibo.client.get(json + page);&#160;&#160; </span>14.  <span>&#160;&#160;&#160; writer.write(response.getResponseAsString());&#160;&#160; </span>15.  <span>} </span><span class="keyword">while</span><span> (page++ * page_count &lt; count);&#160;&#160; </span></span> </div>    

上面这段代码也就将我的微博就扒拉下来了。

**3****、数据解析**

扒拉下来的数据是标准的json格式。但是我一直用不习惯json包，找同学要了一个org.json的，不喜欢用。GSON又要将所有字段都写成实体类，嫌麻烦。所以最后的解析就直接用正则表达式了，简单、暴力、直接。

主要解析了created_at（发布时间）、source（发布客户端）两个字段。获得直接就是用的python的re对象，需要注意的是时间格式为“Tue May 31 17:46:55 +0800 2011”，在Java中需要创建一个格式化对象，格式化代码如下：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <p>SimpleDateFormat **formater** = **new** SimpleDateFormat(

&quot;EEE MMM d HH:mm:ss Z yyyy&quot;, Locale._ENGLISH_);
         </td>       </tr>     </tbody></table> </p>  

格式化之后，最好将Date对象转换成Calender对象，因为Date中的年份不太好用，怎么不好用，自己可以体会一下。

**4****、生成图表**

有了数据，就直接复制到Excel中，生成图表。图表如下：

[![image](/images/2012/03/image_thumb6.png "image")](/images/2012/03/image6.png) 

我是有多爱转发啊……

[![image](/images/2012/03/image_thumb7.png "image")](/images/2012/03/image7.png) 

网页版新浪微博稳居第一，因为我大部分时间还是花在pc上的。买了pad后，在iPad客户端上的发布数应该会上升很多。

[![image](/images/2012/03/image_thumb8.png "image")](/images/2012/03/image8.png) 

一周中，每天发的微博数差不多，也是，我对周几一般来说是没概念的。

[![image](/images/2012/03/image_thumb9.png "image")](/images/2012/03/image9.png) 

一天中，发布的大部分时间都集中在了中午和午夜，夜晚倒能理解，中午为啥也那么高呢？

[![image](/images/2012/03/image_thumb10.png "image")](/images/2012/03/image10.png) 

去年三月份创建的微博帐户，之后发布的微博数急速上升，到了五月到了顶峰，之后就回落。去年十月份之后就比较忙了，一直到了开学，微博又有上升的趋势了。

**5、小结**

以上仅是在无聊中无聊的作品，仅仅是一些字段的孤立的统计，没有任何技术内涵可言，但是还是可以说明一些问题的。不过改天得仔细分析分析我发的微博内容，嗯，是的……
