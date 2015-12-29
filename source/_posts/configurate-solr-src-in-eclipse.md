title: 在Eclipse中配置Solr源码
tags:
  - Eclipse
  - java
  - Solr
id: 735
categories:
  - 技术分享
date: 2013-03-20 00:06:01
---

**1.** 下载solr的src包，并解压

	&nbsp;

	**2.** 解压后，在解压后的根目录执行ant eclipse，即生成eclipse需要的项目文件

	打开eclipse，File &gt; Import &gt; Existing Projects into Workspace

	选择刚才解压后的根目录，这时候java build path等都已经设置好了。

<!--more-->

	&nbsp;

	**3.** Open Type(Ctrl+Shift+T)找到StartSolrJetty 这个类，修改main方法里面的setPort参数为默认的8983，以及ContextPath,War

> War为&rdquo;solr/webapp/web/&rdquo;

	最后的代码应该是这样的：

[code lang="java"]
Server server = new Server();
SocketConnector connector = new SocketConnector();
// Set some timeout options to make debugging easier.
connector.setMaxIdleTime(1000 * 60 * 60);
connector.setSoLingerTime(-1);
connector.setPort(8983); // HWY: MODI
server.setConnectors(new Connector[] { connector });
WebAppContext bb = new WebAppContext();
bb.setServer(server);
bb.setContextPath(&quot;/solr&quot;); // HWY: MODI
bb.setWar(&quot;solr/webapp/web&quot;); // HWY: MODI [/code]

	&nbsp;

	**4.** 设置solr.solr.home，并run

	在run configure中Arguments &gt; VM arguments中写入

> -Dsolr.solr.home=your/path/of/example/solr

	也可以在代码中修改，

> System.setProperty(&quot;solr.solr.home&quot;, &quot;your/path/of/example/solr&quot;);

	使用solr自带的一个example作为sold配置的根目录，也可以设置其他的solr配置目录。点击run即可运行Solr，debug也可以用。

	&nbsp;

	5\. 浏览器输入[http://localhost:8983/solr](http://localhost:8983/solr)查看Solr Admin。

	&nbsp;

> **参考资料：**[http://t.cn/zYduXSq](http://www.zwsun.com/solr_in_eclipse_2012_06_10_post)> 
> 
> 		**BTW： **lucene官网下载镜像地址版本不全，可用这个地址：[http://t.cn/zYDUP9j](http://archive.apache.org/dist/lucene/)
