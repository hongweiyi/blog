title: Hadoop源码 - ipc.RPC
tags:
  - Hadoop
  - RPC
id: 380
categories:
  - 技术分享
date: 2012-02-25 23:00:23
---

**1、前言**

Hadoop是典型的单元数据服务器模型，它将控制流与数据流分离开来，同时两种流的通信机制也不一样，分别为RPC和流式通信，这篇博客主要介绍Hadoop的RPC流程……。关于RPC的简介可以参考：[百度百科](http://baike.baidu.com/view/32726.htm)。
  <!--more-->

Hadoop的RPC主要是通过Java的动态代理（Dynamic Proxy）与反射（Reflect）实现，源代码在org.apache.hadoop.ipc下，有以下几个主要类：

Client：RPC服务的客户端

RPC：实现了一个简单的RPC模型

Server：服务端的抽象类

RPC.Server：服务端的具体类

VersionedProtocol：所有的使用RPC服务的类都要实现该接口，在创建代理时，用来判断代理对象是否创建正确。

**2、通信发生在？**

Hadoop是master-slave模型，master只会接受请求并相应，slave在发送请求的同时，也有可能会接受其它请求，其它请求来自slave伙伴或者client。

VersionedProtocol说了，所有要使用RPC服务的类都要实现该接口，我们可以来看一下有哪些接口继承了该接口。

[![clip_image002](/images/2012/02/clip_image002_thumb3.jpg "clip_image002")](/images/2012/02/clip_image0023.jpg)

**1****）HDFS相关**

**ClientDatanodeProtocol**：client与datanode交互的接口，操作不多，只有一个block恢复的方法。那么，其它数据请求的方法呢？client与datanode主要交互是通过流式的socket实现，源码在DataXceiver，在这里先不说了；

**ClientProtocol**：client与Namenode交互的接口，所有控制流的请求均在这里，如：创建文件、删除文件等；

**DatanodeProtocol**：Datanode与Namenode交互的接口，如心跳、blockreport等；

**NamenodeProtocol**：SecondaryNode与Namenode交互的接口。

**2****）Mapreduce相关**

**InterDatanodeProtocol**：Datanode内部交互的接口，用来更新block的元数据；

**InnerTrackerProtocol**：TaskTracker与JobTracker交互的接口，功能与DatanodeProtocol相似；

**JobSubmissionProtocol**：JobClient与JobTracker交互的接口，用来提交Job、获得Job等与Job相关的操作；

**TaskUmbilicalProtocol**：Task中子进程与母进程交互的接口，子进程即map、reduce等操作，母进程即TaskTracker，该接口可以回报子进程的运行状态（词汇扫盲: umbilical 脐带的, 关系亲密的） 。

**3****）其它**

**AdminOperationProtocol**：不用用户操作的接口，提供一些管理操作，如刷新JobTracker的node列表；

**RefreshAuthorizationPolicyProtocol****，RefreshUserMappingsProtocol：**暂不明白。

**3****、RPC方法**

RPC提供了一个简单的RPC机制，提供以下几种静态方法：

[![clip_image004](/images/2012/02/clip_image004_thumb2.jpg "clip_image004")](/images/2012/02/clip_image0042.jpg)

**1）***Proxy**

waitForProxy、getProxy、stopProxy均是与代理有关的方法，其中wait需要保证namenode启动正常且连接正常，主要由SecondayNode、Datanode、JobTracker使用。

stop方法即停止代理。

get则是一般的获取代理的方法， 创建代理实例，获得代理实例的versioncode，再与getProxy方法传入的versioncode做对比，相同返回代理，不同抛出VersionMismatch异常。

**2）getServer**

创建并返回一个Server实例，由TaskTracker、JobTracker、NameNode、DataNode使用。

**3）call**

静态方法，向一系列服务器发送一系列请求，在源码中没见到那个类使用该方法。但注释提到了：Expert，应该是给系统管理员使用的接口。

**4****、RPC静态类**

RPC方法仅仅提到了方法的作用，但是具体实现没说，具体实现就涉及到了RPC的静态类了，RPC类中有5个静态内部类，分别为：

**RPC.ClientCache****：**用来缓存Client对象；

**RPC.Invocation****：**每次RPC调用传的参数实体类，其中Invocation包括了调用方法（Method）和配置文件；

**RPC.Invoker****：**具体的调用类，采用Java的动态代理机制，继承自InvocationHandler，有remoteId和client成员，id用以标识异步请求对象，client用以调用实现代码；

**RPC.Server****：**org.apache.hadoop.ipc.Server的具体类，实现了抽象类的call方法，获得传入参数的call实例，再获取method方法，调用即可。用的是反射机制，反射很绝，再没使用之前，完全不知道该代码会怎么执行；

**RPC. ****VersionMismatch****：**版本不匹配异常。

**5、小结**

以上是org.apache.hadoop.ipc.RPC类的实现分析，接下来再分析ipc包下的Server与Client类吧！
