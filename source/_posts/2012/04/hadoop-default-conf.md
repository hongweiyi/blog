title: Hadoop默认配置和常用配置
tags:
  - Hadoop
id: 508
categories:
  - 技术分享
date: 2012-04-20 20:48:18
---

**获取默认配置**

配置hadoop，主要是配置core-site.xml，hdfs-site.xml，mapred-site.xml三个配置文件，默认下来，这些配置文件都是空的，所以很难知道这些配置文件有哪些配置可以生效，上网找的配置可能因为各个hadoop版本不同，导致无法生效。浏览更多的配置，有两个方法：

1.选择相应版本的hadoop，下载解压后，找到core-default.xml，hdfs-default.xml，mapred-default.xml。这些分别在hadoop/src/{core | hdfs | mapred}下面。这些就是默认配置，可以参考这些配置的说明和key，配置hadoop集群。

<!--more-->

2.浏览apache官网，三个配置文件链接如下：

[http://hadoop.apache.org/common/docs/r0.20.2/core-default.html](http://hadoop.apache.org/common/docs/r0.20.2/core-default.html)

[http://hadoop.apache.org/common/docs/r0.20.2/hdfs-default.html](http://hadoop.apache.org/common/docs/r0.20.2/hdfs-default.html)

[http://hadoop.apache.org/common/docs/r0.20.2/mapred-default.html](http://hadoop.apache.org/common/docs/r0.20.2/mapred-default.html)

这里是浏览hadoop当前版本号的默认配置文件，其他版本号，要另外去官网找。其中第一个方法找到默认的配置是最好的，因为每个属性都有说明，可以直接使用。另外，core-site.xml是全局配置，hdfs-site.xml和mapred-site.xml分别是hdfs和mapred的局部配置。

**常用的端口配置**

**HDFS****端口**
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td>**参数**</td>
<td>**描述**</td>
<td>**默认**</td>
<td>**例子值**</td>
</tr>
<tr>
<td>fs.default.name</td>
<td>namenode RPC交互端口</td>
<td>8020</td>
<td>hdfs：//master：8020/</td>
</tr>
<tr>
<td>dfs.http.address</td>
<td>NameNode web管理端口</td>
<td>50070</td>
<td>0.0.0.0：50070</td>
</tr>
<tr>
<td>dfs.datanode.address</td>
<td>datanode 控制端口</td>
<td>50010</td>
<td>0.0.0.0：50010</td>
</tr>
<tr>
<td>dfs.datanode.ipc.address</td>
<td>datanode的RPC服务器地址和端口</td>
<td>50020</td>
<td>0.0.0.0：50020</td>
</tr>
<tr>
<td>dfs.datanode.http.address</td>
<td>datanode的HTTP服务器和端口</td>
<td>50075</td>
<td>0.0.0.0：50075</td>
</tr>
</tbody>
</table>
**MR****端口**
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td>**参数**</td>
<td>**描述**</td>
<td>**默认**</td>
<td>**例子值**</td>
</tr>
<tr>
<td>mapred.job.tracker</td>
<td>job tracker交互端口</td>
<td>8021</td>
<td>hdfs：//master：8021/</td>
</tr>
<tr>
<td>mapred.job.tracker.http.address</td>
<td>job tracker的web管理端口</td>
<td>50030</td>
<td>0.0.0.0：50030</td>
</tr>
<tr>
<td>mapred.task.tracker.http.address</td>
<td>task tracker的HTTP端口</td>
<td>50060</td>
<td>0.0.0.0：50060</td>
</tr>
</tbody>
</table>
**其他端口**
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td>**参数**</td>
<td>**描述 **</td>
<td>**默认 **</td>
<td>**例子值**</td>
</tr>
<tr>
<td>dfs.secondary.http.address</td>
<td>secondary NameNode web管理端口</td>
<td>50090</td>
<td>0.0.0.0：28680</td>
</tr>
</tbody>
</table>
**集群目录配置**
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td>**参数**</td>
<td>**描述 **</td>
<td>**默认 **</td>
<td>**例子值**</td>
</tr>
<tr>
<td>dfs.name.dir</td>
<td>name node的元数据，以，号隔开，hdfs会把元数据冗余复制到这些目录，一般这些目录是不同的块设备，不存在的目录会被忽略掉</td>
<td>{hadoop.tmp.dir}

/dfs/name</td>
<td>/hadoop/hdfs/name</td>
</tr>
<tr>
<td>dfs.name.edits.dir</td>
<td>node node的事务文件存储的目录，以，号隔开，hdfs会把事务文件冗余复制到这些目录，一般这些目录是不同的块设备，不存在的目录会被忽略掉</td>
<td>${dfs.name.dir}</td>
<td>${dfs.name.dir}</td>
</tr>
<tr>
<td>fs.checkpoint.dir</td>
<td>secondary NameNode的元数据以，号隔开，hdfs会把元数据冗余复制到这些目录，一般这些目录是不同的块设备，不存在的目录会被忽略掉</td>
<td>${hadoop.tmp.dir}

/dfs/namesecondary</td>
<td>/hadoop/hdfs/

namesecondary</td>
</tr>
<tr>
<td>fs.checkpoint.edits

.dir</td>
<td>secondary NameNode的事务文件存储的目录，以，号隔开，hdfs会把事务文件冗余复制到这些目录</td>
<td>${fs.checkpoint.dir}</td>
<td>${fs.checkpoint.dir}</td>
</tr>
<tr>
<td>hadoop.tmp.dir</td>
<td>临时目录，其他临时目录的父目录</td>
<td>/tmp/hadoop-${user.name}</td>
<td>/hadoop/tmp/hadoop-

${user.name}</td>
</tr>
<tr>
<td>dfs.data.dir</td>
<td>data node的数据目录，以，号隔开，hdfs会把数据存在这些目录下，一般这些目录是不同的块设备，不存在的目录会被忽略掉</td>
<td>${hadoop.tmp.dir}

/dfs/data</td>
<td>/hadoop/hdfs/data1/data

/hadoop/hdfs/data2/data</td>
</tr>
<tr>
<td>mapred.local.dir</td>
<td>MapReduce产生的中间数据存放目录，以，号隔开，hdfs会把数据存在这些目录下，一般这些目录是不同的块设备，不存在的目录会被忽略掉</td>
<td>${hadoop.tmp.dir}

/mapred/local</td>
<td>/hadoop/hdfs/data1/

mapred/local

/hadoop/hdfs/data2/

mapred/local</td>
</tr>
<tr>
<td>mapred.system.dir</td>
<td>MapReduce的控制文件</td>
<td>${hadoop.tmp.dir}

/mapred/system</td>
<td>/hadoop/hdfs/data1/system</td>
</tr>
</tbody>
</table>
**其他配置**
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td>**参数**</td>
<td>**描述 **</td>
<td>**默认 **</td>
<td>**例子值**</td>
</tr>
<tr>
<td>dfs.support.append</td>
<td>支持文件append，主要是支持hbase</td>
<td>false</td>
<td>true</td>
</tr>
<tr>
<td>dfs.replication</td>
<td>文件复制的副本数，如果创建时不指定这个参数，就使用这个默认值作为复制的副本数</td>
<td>3</td>
<td>2</td>
</tr>
</tbody>
</table>
&nbsp;
> 转载：> 
> 
> [http://www.cnblogs.com/ggjucheng/archive/2012/04/17/2454590.html](http://www.cnblogs.com/ggjucheng/archive/2012/04/17/2454590.html)
