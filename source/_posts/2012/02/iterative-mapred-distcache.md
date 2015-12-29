title: 迭代式MapReduce解决方案（二） DistributedCache
tags:
  - Hadoop
  - MapReduce
id: 337
categories:
  - 技术分享
date: 2012-02-23 16:01:52
---

### 1、DistributedCache In Hadoop

此篇文章主要是[前一篇](http://www.hongweiyi.com/?p=250)的后续，主要讲Hadoop的分布式缓存机制的原理与运用。

分布式缓存在MapReduce中称之为DistributedCache，它可以方便map task之间或者reduce task之间共享一些信息，同时也可以将第三方包添加到其classpath路径中去。Hadoop会将缓存数据分发到集群的所有准备启动的节点上，复制到在mapred.temp.dir中配置的目录。

 <!--more-->  

### 2、DistributedCache的使用

DistributedCache的使用的本质其实是添加Configuraton中的属性：mapred.cache.{files|archives}。图方便的话，可以使用DistributedCache类的静态方法。

不省事法：

```
conf.set("mapred.cache.files", "/data/data");
conf.set("mapred.cache. archives", "/data/data.zip");
```

省事法：

* [DistributedCache](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/filecache/DistributedCache.html). `**[addCacheFile](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/filecache/DistributedCache.html#addCacheFile(java.net.URI, org.apache.hadoop.conf.Configuration))**``([URI](http://java.sun.com/javase/6/docs/api/java/net/URI.html?is-external=true),` `[Configuration](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/conf/Configuration.html))`
* [DistributedCache](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/filecache/DistributedCache.html).`**[addArchiveToClassPath](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/filecache/DistributedCache.html#addArchiveToClassPath(org.apache.hadoop.fs.Path, org.apache.hadoop.conf.Configuration, org.apache.hadoop.fs.FileSystem))**``([Path](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/fs/Path.html),` `[Configuration](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/conf/Configuration.html),` `[FileSystem](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/fs/FileSystem.html))`


需要注意的是，上面几行代码需要写在Job类初始化之前，否则在运行会中找不到文件（被折磨了很长时间），因为Job初始化时将传入Configuration对象克隆一份给了JobContext。

在MapReduce的0.21版本以后的org.apache.hadoop.mapreduce均移到org.apache.hadoop.mapred包下。但文档中提供的configure方法是重写的MapReduceBase中的，而新版本中map继承于mapper，reduce继承于reducer，所以configure方法一律改成了setup。要获得cache数据，就得在map/reduce task中的setup方法中取得cache数据，再进行相应操作：  

``` java
@Override
protected void setup(Context context) throws IOException,  
        InterruptedException {  
    super.setup(context);  
    URI[] uris = DistributedCache.getCacheFiles(context  
                .getConfiguration());  
    Path[] paths = DistributedCache.getLocalCacheFiles(context  
                .getConfiguration());  
    // TODO  
}  
```

而三方库的使用稍微简单，只需要将库上传至hdfs，再用代码添加至classpath即可：

```
DistributedCache.addArchiveToClassPath(new Path("/data/test.jar"), conf);
```

### 3、symlink的使用

Symlink其实就是hdfs文件的一个快捷方式，只需要在路径名后加入#linkname，之后在task中使用linkname即使用相应文件，如下：

```
conf.set("mapred.cache.files", "/data/data#mData");
conf.set("mapred.cache. archives", "/data/data.zip#mDataZip");
```

``` java
@Override
protected void setup(Context context) throws IOException,  
        InterruptedException {  
    super.setup(context);  
    FileReader reader = new FileReader(new File("mData"));  
    BufferedReader bReader = new BufferedReader(reader);  
    // TODO  
}
```

在使用symlink之前，需要告知hadoop，如下：

* [DistributedCache.createSymlink(Configuration)](/images/2012/02/DistributedCache.html)

### 4、注意事项

1. 缓存文件（数据、三方库）需上传至HDFS，方能使用；
2. 存较小的情况下，建议将数据全部读入相应节点内存，提高访问速度；
3. 缓存文件是read-only的，不能修改。若要修改得重新输出，将新输出文件作为新缓存进入下一次迭代。
