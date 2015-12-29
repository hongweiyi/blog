title: Java中随机数生成是等概率的吗？
tags:
  - Java
id: 127
categories:
  - 技术分享
date: 2011-03-17 22:32:43
---

今天群里面问了一个问题：
> Java里的随机数生成是等概率的吗？例如：下面这两行代码将从0-99之间随机生成一个数

```java
java.util.Random ran = new java.util.Random();
int ConnectID=ran.nextInt(100);
```
> 问：生成0-99之间任意一个数字的概率是相等的吗？

没怎么用过**Random类**，于是乎随意的浏览了一下**Random类**和**nextInt()方法**源代码，无奈水平有限，短时间内无法参透其中的奥秘啊。所以来了一个快速验证的方法，统计学中不是说过**随机事件**的概率，一般可以通过**大量重复试验**求得其近似值么。于是乎，我也通过**大量重复试验**来求求其概率吧。

<!--more-->

代码如下：

``` java
java.util.Random ran = new java.util.Random();
int connectID;
int[] ids = new int[100];
for (int i = 0; i < 100; i++) {
  ids[i] = 0;
}
for (int i = 0; i < 10000000; i++) {
  connectID = ran.nextInt(100);
  ids[connectID]++;
}
for (int i = 0; i < 100; i++) {
  System.out.println(i + “: “ + ids[i]);
}
```

截一小段结果：

> 0: 99127> 1: 99941
> 2: 100295
> 3: 100187
> 4: 100255
> 5: 100266
> 6: 100280
> 7: 99840
> 8: 100466
> 9: 99616
> 10: 100016

剩下的90个数出现的频率和上面的基本一致，大约为10万次。通过上面的实验，基本可以确定为随机数生成是等概率的
