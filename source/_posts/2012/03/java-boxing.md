title: Java装箱/拆箱技术
tags:
  - Java
id: 409
categories:
  - 技术分享
date: 2012-03-04 01:07:14
---

今天群里提出了一个有趣的问题：

<!--more-->

```
Integer num1 = 128;  
Integer num2 = 128;  
int num3 = 128;  
int num4 = 128;  
System.out.println(num1 == num2);  
System.out.println(num1 == num3);  
System.out.println(num2 == num3);  
System.out.println(num3 == num4);  
System.out.println("============");  
Integer num5 = 12;  
Integer num6 = 12;  
int num7 = 12;  
int num8 = 12;  
System.out.println(num5 == num6);  
System.out.println(num5 == num7);  
System.out.println(num6 == num7);  
System.out.println(num7 == num8);
```

乍一看，好似是常量池的题目，但是运行一遍结果之后，好像有些迷糊，结果只有第一个输出为false，其它地方均为true。

`（num3==num4）`以及`（num7==num8）`这个倒是没什么，常量池基本知识。

Integer与int比较，以及Integer内部比较是怎么样的呢？这就涉及到自动装箱/拆箱技术了，一般来说，具有包装类的语言都有这种技术，如Objective-C、Java等，该技术只是编译器给用户提供的一些方便（编译器糖：Compiler Sugar），那么我们来看看Java是怎么实现的吧。附上字节码：

![image](/images/2012/03/image.png)

Java的自动装箱直接调用了Integer的静态方法Integer.valueOf，实质上还是创建了一个Integer对象。

![image](/images/2012/03/image1.png)

而拆箱呢，有人说是将int变量先装箱，再比较。但是字节码说明了，是获得对象的值，再比较。

还有一个疑问，为什么在128的时候num1 != num2，而改一个常量12就等于了呢？需要说明的是，在自动装箱时对于值从[–128, 127]之间的值，它们被装箱为Integer对象后，会在内存中被重用（装箱对象池？）。即如果创建时值位于界内，它会先判断是否存在该对象，有的话直接指向地址。如果不位于界内，则会直接创建新对象。

这样的功能嘛，还是推荐使用标准的格式（=new Integer(value);），自个明白呢，以后的人也明白。不过自个私底下研究研究还是很开脑的。

> **P.S.**: 反编译字节码命令 javap –c ClassName，不能带.class后缀
