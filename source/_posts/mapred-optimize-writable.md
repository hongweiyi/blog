title: MapReduce优化（二） —— 善用Writable
tags:
  - Hadoop
  - MapReduce
id: 588
categories:
  - 技术分享
date: 2012-09-27 21:44:55
---

**一、简述**

上文主要是数据压缩的角度来分析了MapReduce压缩临时数据的优化，参见：[MapReduce优化（一）](http://www.hongweiyi.com/2012/02/mapred-optimize/)。而这篇会更多的从代码层面说MR任务优化。

MapReduce大多数任务都是做日志分析，而一般的日志分析也就是高级点的WordCount程序：读入一段文本 -&gt; 获取需要的信息 -&gt; 统计输出。
<!--more-->  

我这里的任务会从SequenceFile中读取文档（每个文档4M），每条文档里面有许多行记录，每个记录有一个词和词频。任务需要统计记录中所有词出现的绝对频率以及文档频率。格式如下：   <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <p>word1 freq1\n

word2 freq2\n

word3 freq3\n
         </td>       </tr>     </tbody></table> </p>  

**二、善用****Writable**

程序逻辑应该是这样的：

1）根据换行符分隔文档；

2）读取每一行数据；

3）并输出绝对频率(word, freq)与文档频率(word, ONE)两条记录。

简单明了的程序处理方式应该是这样的：   <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <p>1\. String str = value.toString(); 

2\. String[] ln = str.split(&quot;\n&quot;); 

3\. for(String l : ln) {

4.&#160;&#160;&#160;&#160; String[] words = ln.split(&quot; &quot;);

5.&#160;&#160;&#160;&#160; context.write(new Text(words[0]), new LongWritable(1));

6.&#160;&#160;&#160;&#160; context.write(new Text(words[0]), new LongWritable(Long.parseLong(words[1]));

7\. }
         </td>       </tr>     </tbody></table> </p>  

上面代码可优化的地方有三：

1）在1行代码中，value的toString方法会对数据进行decode，decode效率很慢。而且该方法会分配内存空间；

2）在2行代码中，和split(&quot;\n&quot;)效率低下；

3）5和6行中，每次都会“新”创建“两对”Text，LongWritable对象，GC频繁。

在上面的问题中，问题1和问题2可以一起解决，避免decode和分配内存就需要直接处理byte数组重写Text这个Writable类。只需要继承Text类并添加nextLine()方法即可。添加nextLine()方法是为了逻辑清晰，如果为了更高的效率的话，可以添加nextWord()与nextFreq()方法。

问题3比较常见，在很多资料以及Hadoop自带的Example里面可以看到，输出的键值对均是复用的。用一个全局的KEY和VALUE，直接将新数据set进KEY、VALUE中即可，无需每次新创建相应对象。上面还有一个小地方可以优化的就是，将LongWritable改为IntWritable，减少数据输出。

上面的解决方案似乎还不错了，但是map输出的临时数据依然很大。回查代码，发现context.write()数据太多了，最后的解决方案是将文档频率和绝对频率合并起来，简单点的格式就是：d_freq#freq。这样临时数据一下就减少了一半。

以下是修改后的代码流程：   <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <p>1\. while (value.hasNext()) {

2.&#160;&#160;&#160;&#160; KEY.set(value.nextWord());

3.&#160;&#160;&#160;&#160; VALUE.set(&quot;1#&quot; + value.nextFreq());

4.&#160;&#160;&#160;&#160; context.write(KEY, VALUE);

5\. }
         </td>       </tr>     </tbody></table> </p>  

**三、优化结果**

通过上面的优化，处理300G的数据，单个map task平均时间从3'30''降到45''，FILE_BYTE_WRITEN从1000G降到了450G，任务总时间从4小时降到了30分钟。

P.S.: 20台节点。

**四、后记**

后来我在这个任务里面使用了Gzip压缩，压缩率约为44%。但是对效率的影响太大了，单个map task平均时间从45''升到1'20''，打了一半的折扣，着实让人难以接受。由于对集群没有完全的管理权限，所以无法在这个任务上面尝试Lzo压缩编码，有机会在尝试吧。