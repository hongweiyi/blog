title: Apache Solr —— 常用数据结构（一）
date: 2013-04-08 18:04:00
categories: 技术分享
tags: [Solr, 数据结构]
---

### 1、前言

这一段时间看Solr源码，主要的心思都是放在了整体框架运行流程，或者称之为“算法”。这些流程中间会穿插很多有意思的接口/类，NamedList、DocSet、DocIdSet、Query、Document、FieldType、ResponseBuilder等，这些接口/类看似简单，但是里面又有各种不同的实现，在翻源码的时候，确实会让人有些招架不过来，所以就准备写这篇博文让自己梳理梳理Solr的常用“数据结构”，有了算法和数据结构才能对Solr有一个完整的理解。

<!--more-->

### 2、NamedList

一个简单的键值对列表，与Map不同的是：

* 键名可以重复；
* 保持数据的插入顺序；
* 元素可以通过下标访问；
* 键与值均可为null。

内部实现其实也挺简单，内部容器采用的是ArrayList。list中的内部数据格式为：

* [key1, value1, key2, value2, key3, value3,...]

键与值通过线性排列组合在一起，访问下标则为：

* key = list.get(idx << 1);
* value = list.get((idx << 1) +1);

如需要通过键查询的话，则是遍历list中的所有元素，时间复杂为O(N)。因为设计该类主要是用来实现有序的键值对序列，通过键查询的需求较少。如果需要经常查询的话，可以用SimpleOrderedMap或者直接用Map。

需要说明的是，我并没有看出SimpleOrderedMap有啥名堂，内部实现只是实现了几个构造函数（全部调用super()），和一个clone()。并无其他逻辑。

NamedList主要用来保存各种结果或者参数，变量名常见于res、params、terms。

### 3、DocSet

<center><div style="width: 80%;">![DocSet](/images/sofa-DocSet.png)</div></center>

这DocSet、DocList接口是DocSet数据结构的基础，一般来说DocSet就是用来保存DocId的集合，提供了一些集合的逻辑运算，如：union、intersection、addNot等。

* DocSet是一个无序的Lucene Document Id的集合。
* DocList是一个有序的Lucene Document Id的列表，这里的有序（ordered）应该不是指的排序（sorted），而是docs是顺序有关的，不可随意打乱，因为这个顺序关联一个可选的scores成员。

在DocSet基础上，Solr提供了一个抽象类DocSetBase，DocSet系列中所有的派生类均继承了该方法。该抽象类实现了一些方法，这些方法主要是提供给无该方法实现的子类。如DocSlice，该类就没有实现union方法。

DocSlice就是DocList的实现，好像没有啥特别的地方。由于实现DocList的原因，DocSlice也不建议被修改，它提供了一个subset的方法，可获得其子集。

HashDocSet用的Hash表方法存的docIds，在数据比较少的时候好像比较省内存（但是除了测试代码之外，Solr内部没看到有实际逻辑代码使用了该类- -）

SortedIntDocSet是排序的DocSet，在构造SortedIntDocSet的时候，就已经传入了一个排序好的docs。

BitDocSet是用的位图存放docIds，集合的逻辑运算实现用的lucene的OpenBitSet，这个类也是Solr用的最多的DocSet了。位图的一些知识可以参考我的博文：[《趣味数据结构 - Bitmap》](http://hongweiyi.com/2012/03/data-structure-bitmap/)

> BTW：还有一个DocIdSet，我就觉得稀奇了！DocSet也是放Document Id的，DocIdSet也是放Document Id的。而DocIdSet不直接提供存放Id的容器，只提供了一个迭代器的功能，这样的设计暂时没有弄明白是为什么，先放着吧。

### 4、结尾

今天就只分析NamedList和DocSet了，剩下的等清明小长假后再分析吧，下午出去旅游一下了……
