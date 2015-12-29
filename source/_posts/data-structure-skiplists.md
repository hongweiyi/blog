title: 趣味数据结构 - SkipLists
tags:
  - 数据结构
id: 460
categories:
  - 技术分享
date: 2012-03-09 21:13:11
---

**1****、简介**

给一串有序的数据，如何存储可以增删查改快速方便、扩容简单、实现也简单呢？用数组吧，实现简单，二分法也老快了，但是删除就很麻烦了，且扩容也需要开辟新的空间。用链表吧，新增删除都很快，但是查找就得遍历了。用平衡树（AVL、红黑树）吧，新增删除扩容都很方便，但是实现起来非常麻烦。
  <!--more-->

说完上面的，来推荐一个有趣的数据结构，可方便实现上面几个要求，那就是SkipLists（跳表）。我们先来看看SkipList作者William Pugh对它的定义吧：
  > _Skip lists are a data structure that can be used in place of balanced trees. Skip lists use probabilistic balancing rather than strictly enforced balancing and as a result the algorithms for insertion and deletion in skip lists are much simpler and significantly faster than equivalent algorithms for balanced trees.(1990)_  

SkipLists设计出来就是用来取代平衡树的，SkipList依靠随机的思想，从某种程序上实现了平衡，且插入与删除都比较简单。

**2****、SkipLists思想**

[![image](http://www.hongweiyi.com/wp-content/uploads/2012/03/image_thumb4.png "image")](http://www.hongweiyi.com/wp-content/uploads/2012/03/image4.png) 我们先来看看上面的有序链表（例子Copy自文献），需要查找一个节点，我们得获得头节点，再依次遍历下去，时间复杂度为O(n)。

[![image](http://www.hongweiyi.com/wp-content/uploads/2012/03/image_thumb5.png "image")](http://www.hongweiyi.com/wp-content/uploads/2012/03/image5.png) 如果有上面这种结构，每间隔一个节点添加一个额外指针指向下下个节点。那么每次查找先查下下节点，大于继续查找，小于的话，查找下个节点。这样的话，时间复杂度就降到了O(n/2)。

这基本上就是跳表的核心思想，即通过“空间来换取时间”的一个算法，通过在每个节点中增加了向前的指针，从而提升查找的效率。

**3****、构造过程**

1）给定一个有序的链表；

2）选择链表最上层的最大、最小节点，然后从其它节点中按照一定算法随机选出一些节点，将这些节点组成有序链表。新链表称之为第一层，原链表称之为下一层；

3）为刚选出的每个节点添加一个指针域，这个指针指向下一层中相应的节点。Top指针指向首层首节点；

4）重复2、3步，直到不能选选择出除最大最小节点之外的节点。

如下图：

[![clip_image006](http://www.hongweiyi.com/wp-content/uploads/2012/03/clip_image006_thumb.jpg "clip_image006")](http://www.hongweiyi.com/wp-content/uploads/2012/03/clip_image006.jpg)

**4****、结构特征**

1）一个SkipList应由多个层组成（Level）；

2）SkipList的最底层包含所有的节点；

3）每一层都是一个有序链表；

4）如果节点X出现在第i层，那么所有&lt;i的层都含有X；

5）Top指向最高层的首节点；

6）第i层的节点通过down指针指向下一层相应的节点；

7）每一层均有全局最大最小值。

**5****、其它**

**1****）随机算法**

调用一个随机函数，该函数返回节点会撑到第几层，伪代码如下：    <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <p>randomLevel()

&#160;&#160;&#160; lvl := 1

_&#160;&#160;&#160; -- random_() _that returns a random value in _[0...1)

**&#160;&#160;&#160; while **random() &lt; p **and **lvl &lt; MaxLevel **do**

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; lvl := lvl + 1

**&#160;&#160;&#160; return **lvl
         </td>       </tr>     </tbody></table> </p>  

一般来说，p为0.5，那么能撑到第i的概率为0.5^i。

**2****）查找**

SkipLists查找从Top节点开始，如二分查找一样，大于下一节点就继续，小于则跳到下一层。直到找到节点或者到NULL为止。时间复杂度O(logn)。

**3****）插入**

插入也用上面的查找，但是每下一层均要记录转下的那个节点，将其存入update中。待找到插入的位置之后，需要在所有下层插入节点，根据update节点更新。

同时，插入一个元素也需要调用随机算法，判断是否需要更新链表。使用的随机算法和构造时的算法是一致的。时间复杂度O(logn)。

**4****）删除**

删除找到之后，就直接删除了，没什么特别的地方。时间复杂度O(logn)。
  > 参考资料：
> 
> William Pugh：[Skip lists: a probabilistic alternative to balanced trees](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.85.9211&amp;rep=rep1&amp;type=pdf)
> 
> 跳表：[www.cnblogs.com/xuqiang/archive/2011/05/22/2053516.html](http://www.cnblogs.com/xuqiang/archive/2011/05/22/2053516.html)