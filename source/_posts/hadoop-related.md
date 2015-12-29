title: Hadoop生态图谱
tags:
  - Hadoop
id: 413
categories:
  - 技术分享
date: 2012-03-05 00:21:13
---

当下Hadoop已经成长为一个庞大的体系，貌似只要和海量数据相关的，没有哪个领域缺少Hadoop的身影，下面是一个Hadoop生态系统的图谱，详细的列举了在Hadoop这个生态系统中出现的各种数据工具。
<!--more-->> 1\. 数据抓取系统 － [Nutch](http://nutch.apache.org/) 
> 
> 2\. 海量数据怎么存，当然是用分布式文件系统 － [HDFS](http://hadoop.apache.org/hdfs/) 
> 
> 3\. 数据怎么用呢，分析，处理 - MapReduce框架让你编写代码来实现对大数据的分析工作 
> 
> 4\. 非结构化数据（日志）收集处理 － [fuse](http://fuse.sourceforge.net/),[webdav](http://www.webdav.org/), [chukwa](http://incubator.apache.org/chukwa/), [flume](https://github.com/cloudera/flume/wiki), [Scribe](https://github.com/facebook/scribe) 
> 
> 5\. 数据导入到HDFS中，至此RDBSM也可以加入HDFS的狂欢了 － [Hiho](https://github.com/sonalgoyal/hiho), [sqoop](http://www.cloudera.com/downloads/sqoop/) 
> 
> 6\. MapReduce太麻烦，好吧，让你用熟悉的方式来操作Hadoop里的数据 – [Pig](http://pig.apache.org/), [Hive](http://hive.apache.org/), [Jaql](http://code.google.com/p/jaql/) 
> 
> 7\. 让你的数据可见 － drilldown, [Intellicus](http://www.intellicus.com/) 
> 
> 8\. 用高级语言管理你的任务流 – [oozie](http://yahoo.github.com/oozie/), [Cascading](http://www.cascading.org/) 
> 
> 9\. Hadoop当然也有自己的监控管理工具 – [Hue](https://github.com/cloudera/hue), [karmasphere](http://karmasphere.com/), [eclipse plugin](http://wiki.apache.org/hadoop/EclipsePlugIn), [cacti](http://www.cacti.net/), [ganglia](http://ganglia.sourceforge.net/) 
> 
> 10\. 数据序列化处理与任务调度 – [Avro](http://avro.apache.org/), [Zookeeper](http://zookeeper.apache.org/) 
> 
> 11\. 更多构建在Hadoop上层的服务 – [Mahout](http://mahout.apache.org/), [Elastic map Reduce](http://aws.amazon.com/elasticmapreduce/) 
> 
> 12\. OLTP存储系统 – [Hbase](http://hbase.apache.org/)  

![](http://pic.yupoo.com/iammutex/BKBq9XAQ/POqWJ.png)
  > 转载：
> 
> NoSQLFan：[Hadoop生态图谱](http://blog.nosqlfan.com/html/3675.html)