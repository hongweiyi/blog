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

### 1、Hadoop Pipes简介

Hadoop Pipes是Hadoop MapReduce的C++接口代称。不同于使用标准输入和输出来实现的map代码和reduce代码之间的Streaming编程，Pipes使用Socket作为TaskTracker与C++进程之间数据传输的通道，数据传输为字节流。

<!--more-->

### 2、Hadoop Pipes编程初探

Hadoop Pipes可供开发者编写RecordReader、Mapper、Partitioner、Reducer、RecordWriter五个组件，当然，也可以自定义Combiner。

网上有一大堆Hadoop Pipes的WordCount，个人觉得最好的WordCount还是Hadoop自带的，可以参见目录：$HADOOP_HOME/src/examples/pipes/impl

与Pipes相关的头文件放在了目录：
  > $HADOOP_HOME/c++/Linux-i386oramd64-32/include/hadoop/  

主要的文件为Pipes.hh，该头文件定义了一些抽象类，除去开发者需要编写的五大组件之外，还有JobConf、TaskContext、Closeable、Factory四个。

TaskContext：开发者可以从context中获取当前的key，value，progress和inputSplit等数据信息，当然，比较重要的就是调用emit将结果回传给Hadoop Framework。除了TaskContext，还有MapContext与ReduceContext，代码见下：

``` c
class TaskContext {  
public:  
  class Counter {  
  private:  
    int id;  
  public:  
    Counter(int counterId) : id(counterId) {}  
    Counter(const Counter& counter) : id(counter.id) {}  
    int getId() const { return id; }  
  };  

  virtual const JobConf* getJobConf() = 0;  
  virtual const std::string& getInputKey() = 0;   
  virtual const std::string& getInputValue() = 0;    
  virtual void emit(const std::string& key, const std::string& value) = 0;    
  virtual void progress() = 0;    
  virtual void setStatus(const std::string& status) = 0;  
  virtual Counter*   
getCounter(const std::string& group, const std::string& name) = 0;  
  virtual void incrementCounter(const Counter* counter, uint64_t amount) = 0;  
  virtual ~TaskContext() {}  
};  

class MapContext: public TaskContext {  
public:  
  virtual const std::string& getInputSplit() = 0;  
  virtual const std::string& getInputKeyClass() = 0;  
  virtual const std::string& getInputValueClass() = 0;  
};  

class ReduceContext: public TaskContext {  
public:  
  virtual bool nextValue() = 0;  
};  
```

JobConf：开发者可以通过获得任务的属性

``` c
class JobConf {  
public:  
  virtual bool hasKey(const std::string& key) const = 0;  
  virtual const std::string& get(const std::string& key) const = 0;  
  virtual int getInt(const std::string& key) const = 0;  
  virtual float getFloat(const std::string& key) const = 0;  
  virtual bool getBoolean(const std::string&key) const = 0;  
  virtual ~JobConf() {}  
};  
```

Closeable：这个抽象类是五大组件的基类，只有两个方法，一个close()，一个析构函数。这个设计还是挺有Java风格的。

Factory：一个抽象工厂，用来创建五大组件的类，是模版工厂的基类。具体的可以参见TemplateFactory.hh。开发者在调用runTask时，创建相应的Factory传入即可。

### 3、Hadoop Pipes编程

有了以上的基础知识，就可以开始编写MapReduce任务了。我们可以直接从examples着手，先来看看wordcount-simple.cc。


``` c
// wordcount-simple.cc -> Mapper & Reducer
class WordCountMap: public HadoopPipes::Mapper {  
public:  
  HadoopPipes::TaskContext::Counter* inputWords;  

  WordCountMap(HadoopPipes::TaskContext& context) {  
    inputWords = context.getCounter(WORDCOUNT, INPUT_WORDS);  
  }  

  void map(HadoopPipes::MapContext& context) {  
    std::vector<std::string> words =   
      HadoopUtils::splitString(context.getInputValue(), " ");  
    for(unsigned int i=0; i < words.size(); ++i) {  
      context.emit(words[i], "1");  
    }  
    context.incrementCounter(inputWords, words.size());  
  }  
};  

class WordCountReduce: public HadoopPipes::Reducer {  
public:  
  HadoopPipes::TaskContext::Counter* outputWords;  

  WordCountReduce(HadoopPipes::TaskContext& context) {  
    outputWords = context.getCounter(WORDCOUNT, OUTPUT_WORDS);  
  }  

  void reduce(HadoopPipes::ReduceContext& context) {  
    int sum = 0;  
    while (context.nextValue()) {  
      sum += HadoopUtils::toInt(context.getInputValue());  
    }  
    context.emit(context.getInputKey(), HadoopUtils::toString(sum));  
    context.incrementCounter(outputWords, 1);   
  }  
};
```  

该任务编写了两个主要组件，mapper与reducer。要实现这两个组件需要继承相应的基类。基类声明如下：

``` c
class Mapper: public Closable {  
public:  
  virtual void map(MapContext& context) = 0;  
};  

class Reducer: public Closable {  
public:  
  virtual void reduce(ReduceContext& context) = 0;  
};  
```

继承了相应的基类，就可以大胆的通过context获得key/value实现自己的逻辑了，结果处理完毕后，需要通过context.emit(key, value)将结果发送到下一阶段。

注：

1. 由于Factory创建对象需要传入Context对象，所以还需要实现一个构造函数，参数为TaskContext。
2. Hadoop Pipes内部规定，map与reduce的key/value均为Text类型，在C++中表现为string类型。不过，Hadoop还是做得比较贴心，有专门的方法负责处理string，具体可以参见StringUtils.hh。
3. Counter可以称之为统计器，可供开发者统计一些需要的数据，如读入行数、处理字节数等。任务完毕后，可以在web控制参看结果。

```
// wordcount-part.cc -> Partitioner
class WordCountPartitioner: public HadoopPipes::Partitioner {
    public:
    WordCountPartitioner(HadoopPipes::TaskContext& context){}
         virtual int partition(const std::string& key, int numOfReduces) {   
           return 0;
        }
};  
```

该实例在提供简单Mapper与Reducer方法的同时，还提供了Partitioner，实例实现较为简单，直接返回了第一个reduce位置。开发者自定义的Partitioner同mapper/reducer一致，需要继承其基类HadoopPipes:: RecordWriter，也需要提供一个传入TaskContext的构造函数，它的声明如下：

``` c
class Partitioner {   
   public:   
    virtual int partition(const std::string& key, int numOfReduces) = 0;   
    virtual ~Partitioner() {}   
};
```

Partitioner编写方法与Java的一致，对于partition方法，框架会自动为它传入两个参数，分别为key值和reduce task的个数numOfReduces，用户只需返回一个0~ numOfReduces-1的值即可。

wordcount-nopipe.cc -> RecordReader & RecordWriter

这个实例的命名让我思考了很久，是nopipe还是nopart呢？该实例没有实现Partitioner，实现了RecordReader与RecordWriter。框架在运行之初，检查到开发者没有使用Java内置的RecordWriter，所以就只将InputSplit信息通过Pipes发送给C++ Task，由Task实现自身的Record读方法。同样，在Record写数据时任务也没走Pipes，直接将数据写到了相应的位置，写临时文件会直接写到磁盘，写HDFS则需要通过libhdfs进行写操作。具体Pipes运行流程，请参见下篇博文。

RecordReader/RecordWriter实现较长，这里就不贴了，贴一下这俩的基类：

```
class RecordReader: public Closable {   
   public:   
   virtual bool next(std::string& key, std::string& value) = 0;   4.   // 读进度   
      virtual float getProgress() = 0;   
};   
class RecordWriter: public Closable {   
   public:   
   virtual void emit(const std::string& key,
                     const std::string& value) = 0;
};
```    

对于RecordReader，用户自定义的构造函数需携带类型为HadoopPipes::MapContext的参数（而不能是TaskContext），通过该参数的getInputSplit()的方法，用户可以获取经过序列化的InpuSplit对象，Java端采用不同的InputFormat可导致InputSplit对象格式不同，但对于大多数InpuSplit对象，它们可以提供至少三个信息：当前要处理的InputSplit所在的文件名，所在文件中的偏移量，它的长度。用户获取这三个信息后，可使用libhdfs库读取文件，以实现next方法。

用户自定的RecordWriter的构造函数需携带参数TaskContext，通过该参数的getJobConf()可获取一个HadoopPipes::JobConf的对象，用户可从该对象中获取该reduce task的各种参数，如：该reduce task的编号（这对于确定输出文件名有用），reduce task的输出目录等。同时实现emit方法，将数据写入文件。

### 4、Hadoop Pipes任务提交

Hadoop Pipes任务提交命令根据Hadoop版本而不一，主体的命令有如下：

```
hadoop pipes [-conf <path>] [-D <key=value>, <key=value>, …] [-input <path>] [-output <path>] [-jar <jar file>] [-inputformat <class>] [-map <class>] [-partitioner <class>] [-reduce <class>] [-writer <class>] [-program <executable>]
```

想使用其它静态数据的话，还可以使用-files命令，该命令就是DistributedCache，直接将静态数据分发到所有datanode上。具体机制参见：[DistributedCache](http://www.hongweiyi.com/2012/02/iterative-mapred-distcache/)。使用如下：

> shell: bin/hadoop pipes … -files dict.txt
>
> c: file = fopen(“dict.txt”, “r”); // 直接根据文件名读取  

### 5、小结

本篇博文简要了说了一下Hadoop Pipes的使用方法，下篇博文会对Hadoop Pipes的运行机制进行一个深入的讲解。

在这里贴一下董的优化意见：为了提高系能，RecordReader和RecordWriter最好采用Java代码实现（或者重用Hadoop中自带的），这是因为Hadoop自带的C++库libhdfs采用JNI实现，底层还是要调用Java相关接口，效率很低，此外，如果要处理的文件为二进制文件或者其他非文本文件，libhdfs可能不好处理。

> 参考资料：
>
> 董的博客: [Hadoop pipes编程](http://dongxicheng.org/mapreduce/hadoop-pipes-programming/)
>
> 《Hadoop权威指南》
