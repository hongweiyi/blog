title: Hadoop Pipes运行机制
tags:
  - Hadoop
  - Hadoop Pipes
  - MapReduce
id: 541
categories:
  - 技术分享
date: 2012-05-13 14:20:19
---

**1****、前言**

Hadoop Pipes可供C++开发者开发MapReduce任务。文献与书籍上也写了，C++与Java是通过Socket通信，但是具体的运行机制是什么还是得参考源码。

这篇博文主要从源码角度来讲解Hadoop Pipes运行机制以及设计原理，实际的Hadoop Pipes编程请参见：**[Hadoop Pipes编程](http://www.hongweiyi.com/2012/05/hadoop-pipes/)**

<!--more-->

**2****、Hadoop Pipes****运行图解**

[![image](http://www.hongweiyi.com/wp-content/uploads/2012/05/image_thumb.png "image")](http://www.hongweiyi.com/wp-content/uploads/2012/05/image.png) 

**3****、Hadoop****运行机制**

Hadoop端主要类均在org.apache.hadoop.mapred.pipes包下，见下图。

其中，Application是JVM中主要运行程序，PipesMapRunner、PipesReducer、PipesPartitioner、PipesNonJavaInputFormat分别对应C++版的Mapper、Reducer、Partitioner、RecordReader，由于重写RecordWriter后，C++会直接写文件，这里就没有对应的类了。DownwardProtocol/BinaryProtocol、UpwardProtocol/OutputProtocol是Java与C++交互的接口代理类。

[![image](http://www.hongweiyi.com/wp-content/uploads/2012/05/image_thumb1.png "image")](http://www.hongweiyi.com/wp-content/uploads/2012/05/image1.png)

开发者通过$HADOOP_HOME/bin/hadoop pipes将作业提交到了包下的Submitter类。运行过程就直接贴文字了，可以结合代码一起看：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <p>1 解析命令行参数

2 setupPipes(job)

2.1 设置Mapper，Partitioner，Reducer，RecordWriter，如果不是java编写的，则用PipesMapRunner，PipesPartitioner，PipesReducer，NullOutputFormat（所有输出均输出到/dev/null中）；

2.2 设置map/reduce的key/value class，均为Text.class；

2.3 设置RecordReader，如果不是java编写的，则用PipesNonJavaInputFormat；

2.4 获得运行程序，debug脚本以及缓存文件；

3 JobClient.submitJob(job);
         </td>       </tr>     </tbody></table> </p>  

JobClient提交任务和非Pipes编程提交过程一致，进行Task调度分配之后，就会在分配的TaskTracker上开启JVM进程，运行Runner。这里解析一下PipesMapRunner的运行机制：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <p>1 创建Application

1.1 创建ServerSocket

1.2 设置环境（临时文件位置、命令端口等）

1.3 获得执行文件，并设置执行权限（chmod +x）

1.4 执行任务，通过java.lang.ProcessBuilder

1.5 创建任务交互代理，DownwardProcotol对象。

1.5.1 创建接收交互代理，UplinkReaderThread对象

1.5.2 循环接受客户端的请求

1.6 downlink.start()，发送消息，客户端可以开始运行

2 如果不是Java编写的RecordReader，直接发送一个InputSplit（注：只是Split的信息，不包括文件数据）给客户端；反之，发送InputSplit之后，再循环读取split，将record格式化之后，将KVP发给客户端。
         </td>       </tr>     </tbody></table> </p>  

PipesReducer与PipesMapRunner运行机制一致，见下：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <p>1 创建Application，与Map一致；

2 向客户端发送key，之后循环发送value。
         </td>       </tr>     </tbody></table> </p>  

以上是Hadoop端的运行机制，C++端的与Java的也基本一致，源文件在$HADOOP_HOME/src/c++/pipes/impl/HadoopPipes.cc

在组件运行时，会用ProcessBuilder运行C++可执行文件，可执行文件的main程序基本上都是这样写的： 
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span class="datatypes">int</span><span> main(</span><span class="datatypes">int</span><span> argc, </span><span class="datatypes">char</span><span> *argv[]) {&#160;&#160; </span></span>2.  <span>&#160; </span><span class="keyword">return</span><span> HadoopPipes::runTask(HadoopPipes::TemplateFactory&lt;WordCountMap,&#160;&#160;&#160; </span></span>3.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; WordCountReduce&gt;());&#160;&#160; </span>4.  <span>}&#160;&#160; </span> </div>    

调用了HadoopUtils::runTask(factory)方法，运行机制如下：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <p>1 创建运行环境；

2 获得socket端口，如果有端口，则创建socket，并获得其输入输出流。如果没有端口，则获得文件输出输入流；

3 创建ping线程，该线程每隔5s发送一次心跳信息；

4 等待接受任务；

5 循环获得任务消息，直到结束（done）；

6 通知Hadoop完成任务，关闭流
         </td>       </tr>     </tbody></table> </p>  

**4****、Hadoop Pipes****浅析**

Hadoop Pipes采用类RPC机制，封装了Hadoop端与C++端的调用接口。Hadoop调用C++的协议为DownwardProtocol，C++调用Hadoop的为UpwardProtocol。同时也封装了传输数据序列化的接口（SerialUtils.cc），代码结构十分清晰。

但是实际使用中也有一定缺陷，调试起来十分麻烦。C++端挂了之后，Hadoop也就接受不到的心跳消息，所以错误一律为：Pipes Broken。Apache的维基上有一个条目：[howToDebugMapReducePrograms](http://wiki.apache.org/hadoop/HowToDebugMapReducePrograms)，改天得好好研究一下。

&#160;

  > **参考资料：**
> 
> [董的博客](http://dongxicheng.org/mapreduce/hadoop-pipes-architecture/)
> 
> Hadoop源码