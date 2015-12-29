title: Hadoop Pipes编程
tags:
  - Hadoop
  - Hadoop Pipes
  - MapReduce
id: 533
categories:
  - 技术分享
date: 2012-05-12 23:04:42
---

**1****、Hadoop Pipes****简介**

Hadoop Pipes是Hadoop MapReduce的C++接口代称。不同于使用标准输入和输出来实现的map代码和reduce代码之间的Streaming编程，Pipes使用Socket作为TaskTracker与C++进程之间数据传输的通道，数据传输为字节流。

<!--more-->

**2****、Hadoop Pipes****编程初探**

Hadoop Pipes可供开发者编写RecordReader、Mapper、Partitioner、Reducer、RecordWriter五个组件，当然，也可以自定义Combiner。

网上有一大堆Hadoop Pipes的WordCount，个人觉得最好的WordCount还是Hadoop自带的，可以参见目录：$HADOOP_HOME/src/examples/pipes/impl

与Pipes相关的头文件放在了目录：
  > $HADOOP_HOME/c++/Linux-i386oramd64-32/include/hadoop/  

主要的文件为Pipes.hh，该头文件定义了一些抽象类，除去开发者需要编写的五大组件之外，还有JobConf、TaskContext、Closeable、Factory四个。

**TaskContext****：**开发者可以从context中获取当前的key，value，progress和inputSplit等数据信息，当然，比较重要的就是调用emit将结果回传给Hadoop Framework。除了TaskContext，还有MapContext与ReduceContext，代码见下： 
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span class="keyword">class</span><span> TaskContext {&#160;&#160; </span></span>
2.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>
3.  <span>&#160; </span><span class="keyword">class</span><span> Counter {&#160;&#160; </span></span>
4.  <span>&#160; </span><span class="keyword">private</span><span>:&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160; </span><span class="datatypes">int</span><span> id;&#160;&#160; </span></span>
6.  <span>&#160; </span><span class="keyword">public</span><span>:&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160; Counter(</span><span class="datatypes">int</span><span> counterId) : id(counterId) {}&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160; Counter(</span><span class="keyword">const</span><span> Counter&amp; counter) : id(counter.id) {}&#160;&#160; </span></span>
9.  <span>&#160;&#160;&#160; </span><span class="datatypes">int</span><span> getId() </span><span class="keyword">const</span><span> { </span><span class="keyword">return</span><span> id; }&#160;&#160; </span></span>
10.  <span>&#160; };&#160;&#160; </span>
11.  <span>&#160;&#160;&#160;&#160; </span>
12.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">const</span><span> JobConf* getJobConf() = 0;&#160;&#160; </span></span>
13.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">const</span><span> std::string&amp; getInputKey() = 0;&#160;&#160;&#160; </span></span>
14.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">const</span><span> std::string&amp; getInputValue() = 0;&#160;&#160;&#160;&#160; </span></span>
15.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">void</span><span> emit(</span><span class="keyword">const</span><span> std::string&amp; key, </span><span class="keyword">const</span><span> std::string&amp; value) = 0;&#160;&#160;&#160;&#160; </span></span>
16.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">void</span><span> progress() = 0;&#160;&#160;&#160;&#160; </span></span>
17.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">void</span><span> setStatus(</span><span class="keyword">const</span><span> std::string&amp; status) = 0;&#160;&#160; </span></span>
18.  <span>&#160; </span><span class="keyword">virtual</span><span> Counter*&#160;&#160;&#160; </span></span>
19.  <span>getCounter(</span><span class="keyword">const</span><span> std::string&amp; group, </span><span class="keyword">const</span><span> std::string&amp; name) = 0;&#160;&#160; </span></span>
20.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">void</span><span> incrementCounter(</span><span class="keyword">const</span><span> Counter* counter, uint64_t amount) = 0;&#160;&#160; </span></span>
21.  <span>&#160; </span><span class="keyword">virtual</span><span> ~TaskContext() {}&#160;&#160; </span></span>
22.  <span>};&#160;&#160; </span>
23.  <span>&#160; </span>
24.  <span></span><span class="keyword">class</span><span> MapContext: </span><span class="keyword">public</span><span> TaskContext {&#160;&#160; </span></span>
25.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>
26.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">const</span><span> std::string&amp; getInputSplit() = 0;&#160;&#160; </span></span>
27.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">const</span><span> std::string&amp; getInputKeyClass() = 0;&#160;&#160; </span></span>
28.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">const</span><span> std::string&amp; getInputValueClass() = 0;&#160;&#160; </span></span>
29.  <span>};&#160;&#160; </span>
30.  <span>&#160; </span>
31.  <span></span><span class="keyword">class</span><span> ReduceContext: </span><span class="keyword">public</span><span> TaskContext {&#160;&#160; </span></span>
32.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>
33.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="datatypes">bool</span><span> nextValue() = 0;&#160;&#160; </span></span>
34.  <span>};&#160;&#160; </span> </div>    

**JobConf**：开发者可以通过获得任务的属性 
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span class="keyword">class</span><span> JobConf {&#160;&#160; </span></span>
2.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>
3.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="datatypes">bool</span><span> hasKey(</span><span class="keyword">const</span><span> std::string&amp; key) </span><span class="keyword">const</span><span> = 0;&#160;&#160; </span></span>
4.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">const</span><span> std::string&amp; get(</span><span class="keyword">const</span><span> std::string&amp; key) </span><span class="keyword">const</span><span> = 0;&#160;&#160; </span></span>
5.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="datatypes">int</span><span> getInt(</span><span class="keyword">const</span><span> std::string&amp; key) </span><span class="keyword">const</span><span> = 0;&#160;&#160; </span></span>
6.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="datatypes">float</span><span> getFloat(</span><span class="keyword">const</span><span> std::string&amp; key) </span><span class="keyword">const</span><span> = 0;&#160;&#160; </span></span>
7.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="datatypes">bool</span><span> getBoolean(</span><span class="keyword">const</span><span> std::string&amp;key) </span><span class="keyword">const</span><span> = 0;&#160;&#160; </span></span>
8.  <span>&#160; </span><span class="keyword">virtual</span><span> ~JobConf() {}&#160;&#160; </span></span>
9.  <span>};&#160;&#160; </span> </div>    

**Closeable**：这个抽象类是五大组件的基类，只有两个方法，一个close()，一个析构函数。这个设计还是挺有Java风格的。

**Factory**：一个抽象工厂，用来创建五大组件的类，是模版工厂的基类。具体的可以参见TemplateFactory.hh。开发者在调用runTask时，创建相应的Factory传入即可。

**3****、Hadoop Pipes****编程**

有了以上的基础知识，就可以开始编写MapReduce任务了。我们可以直接从examples着手，先来看看wordcount-simple.cc。

**wordcount-simple.cc -&gt; Mapper &amp; Reducer** 
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span class="keyword">class</span><span> WordCountMap: </span><span class="keyword">public</span><span> HadoopPipes::Mapper {&#160;&#160; </span></span>
2.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>
3.  <span>&#160; HadoopPipes::TaskContext::Counter* inputWords;&#160;&#160; </span>
4.  <span>&#160;&#160;&#160;&#160; </span>
5.  <span>&#160; WordCountMap(HadoopPipes::TaskContext&amp; context) {&#160;&#160; </span>
6.  <span>&#160;&#160;&#160; inputWords = context.getCounter(WORDCOUNT, INPUT_WORDS);&#160;&#160; </span>
7.  <span>&#160; }&#160;&#160; </span>
8.  <span>&#160;&#160;&#160;&#160; </span>
9.  <span>&#160; </span><span class="keyword">void</span><span> map(HadoopPipes::MapContext&amp; context) {&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160; std::vector&lt;std::string&gt; words =&#160;&#160;&#160; </span>
11.  <span>&#160;&#160;&#160;&#160;&#160; HadoopUtils::splitString(context.getInputValue(), </span><span class="string">&quot; &quot;</span><span>);&#160;&#160; </span></span>
12.  <span>&#160;&#160;&#160; </span><span class="keyword">for</span><span>(unsigned </span><span class="datatypes">int</span><span> i=0; i &lt; words.size(); ++i) {&#160;&#160; </span></span>
13.  <span>&#160;&#160;&#160;&#160;&#160; context.emit(words[i], </span><span class="string">&quot;1&quot;</span><span>);&#160;&#160; </span></span>
14.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
15.  <span>&#160;&#160;&#160; context.incrementCounter(inputWords, words.size());&#160;&#160; </span>
16.  <span>&#160; }&#160;&#160; </span>
17.  <span>};&#160;&#160; </span>
18.  <span>&#160; </span>
19.  <span></span><span class="keyword">class</span><span> WordCountReduce: </span><span class="keyword">public</span><span> HadoopPipes::Reducer {&#160;&#160; </span></span>
20.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>
21.  <span>&#160; HadoopPipes::TaskContext::Counter* outputWords;&#160;&#160; </span>
22.  <span>&#160; </span>
23.  <span>&#160; WordCountReduce(HadoopPipes::TaskContext&amp; context) {&#160;&#160; </span>
24.  <span>&#160;&#160;&#160; outputWords = context.getCounter(WORDCOUNT, OUTPUT_WORDS);&#160;&#160; </span>
25.  <span>&#160; }&#160;&#160; </span>
26.  <span>&#160; </span>
27.  <span>&#160; </span><span class="keyword">void</span><span> reduce(HadoopPipes::ReduceContext&amp; context) {&#160;&#160; </span></span>
28.  <span>&#160;&#160;&#160; </span><span class="datatypes">int</span><span> sum = 0;&#160;&#160; </span></span>
29.  <span>&#160;&#160;&#160; </span><span class="keyword">while</span><span> (context.nextValue()) {&#160;&#160; </span></span>
30.  <span>&#160;&#160;&#160;&#160;&#160; sum += HadoopUtils::toInt(context.getInputValue());&#160;&#160; </span>
31.  <span>&#160;&#160;&#160; }&#160;&#160; </span>
32.  <span>&#160;&#160;&#160; context.emit(context.getInputKey(), HadoopUtils::toString(sum));&#160;&#160; </span>
33.  <span>&#160;&#160;&#160; context.incrementCounter(outputWords, 1);&#160;&#160;&#160; </span>
34.  <span>&#160; }&#160;&#160; </span>
35.  <span>};&#160; </span> </div>    

该任务编写了两个主要组件，mapper与reducer。要实现这两个组件需要继承相应的基类。基类声明如下： 
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span class="keyword">class</span><span> Mapper: </span><span class="keyword">public</span><span> Closable {&#160;&#160; </span></span>
2.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>
3.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">void</span><span> map(MapContext&amp; context) = 0;&#160;&#160; </span></span>
4.  <span>};&#160;&#160; </span>
5.  <span>&#160; </span>
6.  <span></span><span class="keyword">class</span><span> Reducer: </span><span class="keyword">public</span><span> Closable {&#160;&#160; </span></span>
7.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>
8.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">void</span><span> reduce(ReduceContext&amp; context) = 0;&#160;&#160; </span></span>
9.  <span>};&#160;&#160; </span> </div>    

继承了相应的基类，就可以大胆的通过context获得key/value实现自己的逻辑了，结果处理完毕后，需要通过context.emit(key, value)将结果发送到下一阶段。

注：

1）由于Factory创建对象需要传入Context对象，所以还需要实现一个构造函数，参数为TaskContext。

2）Hadoop Pipes内部规定，map与reduce的key/value均为Text类型，在C++中表现为string类型。不过，Hadoop还是做得比较贴心，有专门的方法负责处理string，具体可以参见StringUtils.hh。

3）Counter可以称之为统计器，可供开发者统计一些需要的数据，如读入行数、处理字节数等。任务完毕后，可以在web控制参看结果。

**wordcount-part.cc -&gt; Partitioner** 
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span class="keyword">class</span><span> WordCountPartitioner: </span><span class="keyword">public</span><span> HadoopPipes::Partitioner {&#160;&#160; </span></span>2.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>3.  <span>&#160; WordCountPartitioner(HadoopPipes::TaskContext&amp; context){}&#160;&#160; </span>4.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="datatypes">int</span><span> partition(</span><span class="keyword">const</span><span> std::string&amp; key, </span><span class="datatypes">int</span><span> numOfReduces) {&#160;&#160; </span></span>5.  <span>&#160;&#160;&#160; </span><span class="keyword">return</span><span> 0;&#160;&#160; </span></span>6.  <span>&#160; }&#160;&#160; </span>7.  <span>};&#160;&#160; </span> </div>    

该实例在提供简单Mapper与Reducer方法的同时，还提供了Partitioner，实例实现较为简单，直接返回了第一个reduce位置。开发者自定义的Partitioner同mapper/reducer一致，需要继承其基类HadoopPipes:: RecordWriter，也需要提供一个传入TaskContext的构造函数，它的声明如下： 
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span class="keyword">class</span><span> Partitioner {&#160;&#160; </span></span>2.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>3.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="datatypes">int</span><span> partition(</span><span class="keyword">const</span><span> std::string&amp; key, </span><span class="datatypes">int</span><span> numOfReduces) = 0;&#160;&#160; </span></span>4.  <span>&#160; </span><span class="keyword">virtual</span><span> ~Partitioner() {}&#160;&#160; </span></span>5.  <span>};&#160; </span> </div>    

Partitioner编写方法与Java的一致，对于partition方法，框架会自动为它传入两个参数，分别为key值和reduce task的个数numOfReduces，用户只需返回一个0~ numOfReduces-1的值即可。

**wordcount-nopipe.cc -&gt; RecordReader &amp; RecordWriter**

这个实例的命名让我思考了很久，是nopipe还是nopart呢？该实例没有实现Partitioner，实现了RecordReader与RecordWriter。框架在运行之初，检查到开发者没有使用Java内置的RecordWriter，所以就只将InputSplit信息通过Pipes发送给C++ Task，由Task实现自身的Record读方法。同样，在Record写数据时任务也没走Pipes，直接将数据写到了相应的位置，写临时文件会直接写到磁盘，写HDFS则需要通过libhdfs进行写操作。具体Pipes运行流程，请参见下篇博文。

RecordReader/RecordWriter实现较长，这里就不贴了，贴一下这俩的基类： 
  <div class="dp-highlighter">   <div class="bar"></div>    

1.  <span><span class="keyword">class</span><span> RecordReader: </span><span class="keyword">public</span><span> Closable {&#160;&#160; </span></span>2.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>3.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="datatypes">bool</span><span> next(std::string&amp; key, std::string&amp; value) = 0;&#160;&#160; </span></span>4.  <span>&#160; </span><span class="comment">// 读进度 </span><span>&#160; </span></span>5.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="datatypes">float</span><span> getProgress() = 0;&#160;&#160; </span></span>6.  <span>};&#160;&#160; </span>7.  <span>&#160; </span>8.  <span></span><span class="keyword">class</span><span> RecordWriter: </span><span class="keyword">public</span><span> Closable {&#160;&#160; </span></span>9.  <span></span><span class="keyword">public</span><span>:&#160;&#160; </span></span>10.  <span>&#160; </span><span class="keyword">virtual</span><span>&#160;</span><span class="keyword">void</span><span> emit(</span><span class="keyword">const</span><span> std::string&amp; key,&#160;&#160; </span></span>11.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">const</span><span> std::string&amp; value) = 0;&#160;&#160; </span></span>12.  <span>};&#160; </span> </div>    

对于RecordReader，用户自定义的构造函数需携带类型为HadoopPipes::MapContext的参数（而不能是TaskContext），通过该参数的getInputSplit()的方法，用户可以获取经过序列化的InpuSplit对象，Java端采用不同的InputFormat可导致InputSplit对象格式不同，但对于大多数InpuSplit对象，它们可以提供至少三个信息：当前要处理的InputSplit所在的文件名，所在文件中的偏移量，它的长度。用户获取这三个信息后，可使用libhdfs库读取文件，以实现next方法。

用户自定的RecordWriter的构造函数需携带参数TaskContext，通过该参数的getJobConf()可获取一个HadoopPipes::JobConf的对象，用户可从该对象中获取该reduce task的各种参数，如：该reduce task的编号（这对于确定输出文件名有用），reduce task的输出目录等。同时实现emit方法，将数据写入文件。

**4****、Hadoop Pipes****任务提交**

Hadoop Pipes任务提交命令根据Hadoop版本而不一，主体的命令有如下：

hadoop pipes [-conf &lt;path&gt;] [-D &lt;key=value&gt;, &lt;key=value&gt;, ...] [-input &lt;path&gt;] [-output &lt;path&gt;] [-jar &lt;jar file&gt;] [-inputformat &lt;class&gt;] [-map &lt;class&gt;] [-partitioner &lt;class&gt;] [-reduce &lt;class&gt;] [-writer &lt;class&gt;] [-program &lt;executable&gt;]    <table border="0" cellspacing="1" cellpadding="0"><tbody>       <tr>         <td valign="top" width="254">           <p>**命**** ****令**
         </td>          <td valign="top" width="292">           

**描**** ****述**
         </td>       </tr>        <tr>         <td valign="top" width="254">           

-conf &lt;path&gt;
         </td>          <td valign="top" width="292">           

任务配置文件
         </td>       </tr>        <tr>         <td valign="top" width="254">           

-D &lt;key=value&gt;
         </td>          <td valign="top" width="292">           

添加单独的配置
         </td>       </tr>        <tr>         <td valign="top" width="254">           

-input &lt;path&gt;
         </td>          <td valign="top" width="292">           

输入数据目录
         </td>       </tr>        <tr>         <td valign="top" width="254">           

-output &lt;path&gt;
         </td>          <td valign="top" width="292">           

输出数据目录
         </td>       </tr>        <tr>         <td valign="top" width="254">           

-jar &lt;jar file&gt;
         </td>          <td valign="top" width="292">           

应用程序jar包
         </td>       </tr>        <tr>         <td valign="top" width="254">           

-inputformat class
         </td>          <td valign="top" width="292">           

#Java版的InputFormat
         </td>       </tr>        <tr>         <td valign="top" width="254">           

-map &lt;class&gt;
         </td>          <td valign="top" width="292">           

Java版的Mapper
         </td>       </tr>        <tr>         <td valign="top" width="254">           

-partitioner &lt;class&gt;
         </td>          <td valign="top" width="292">           

Java版的Partitioner
         </td>       </tr>        <tr>         <td valign="top" width="254">           

-reduce &lt;class&gt;
         </td>          <td valign="top" width="292">           

Java版的Reducer
         </td>       </tr>        <tr>         <td valign="top" width="254">           

-writer &lt;class&gt;
         </td>          <td valign="top" width="292">           

Java版的 RecordWriter
         </td>       </tr>        <tr>         <td valign="top" width="254">           

-program &lt;executable&gt;
         </td>          <td valign="top" width="292">           

C++可执行程序
         </td>       </tr>     </tbody></table> </p>  

想使用其它静态数据的话，还可以使用-files命令，该命令就是DistributedCache，直接将静态数据分发到所有datanode上。具体机制参见：[DistributedCache](http://www.hongweiyi.com/2012/02/iterative-mapred-distcache/)。使用如下：
  > **shell**: bin/hadoop pipes … -files dict.txt
> 
> **c**: file = fopen(“dict.txt”, “r”); // 直接根据文件名读取  

**5****、小结**

本篇博文简要了说了一下Hadoop Pipes的使用方法，下篇博文会对Hadoop Pipes的运行机制进行一个深入的讲解。

在这里贴一下董的优化意见：为了提高系能，RecordReader和RecordWriter最好采用Java代码实现（或者重用Hadoop中自带的），这是因为Hadoop自带的C++库libhdfs采用JNI实现，底层还是要调用Java相关接口，效率很低，此外，如果要处理的文件为二进制文件或者其他非文本文件，libhdfs可能不好处理。

&#160;
  > **参考资料：**
> 
> 董的博客: [Hadoop pipes编程](http://dongxicheng.org/mapreduce/hadoop-pipes-programming/)
> 
> 《Hadoop权威指南》
