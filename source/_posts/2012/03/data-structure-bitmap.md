title: 趣味数据结构 - BitMap
tags:
  - 数据结构
id: 422
categories:
  - 技术分享
date: 2012-03-05 21:37:49
---

**1****、什么是Bit-Map**

Bit-Map被译为位图，和人讨论的时候，常常会与.BMP搞混，这个Map我觉得翻译成映射更为合适，Bit-Map也算是Hash的一直极致运用吧。Bit-Map会用Bit来标记某个元素对应的value，如何标记的呢，见下例：
  <!--more-->

我们现在有(1,2,5,8,10)数组，常规来说是这样声明的：

int[] array = {1, 2, 5, 8, 10}

上面这样声明会占用4×5个字节，即20个字节，少量数据可能没有什么特别大的感觉，如果数组长度为10,000,000，这样的方式就会占用4G的内存。

如果用Bit-Map的话，可以这样来组织：

byte[] bytes = new bytes[2];

bytes[0] = 01100100; // 就直接写二进制了

bytes[1] = 10100000;

**2****、Bit-Map建立**

有了上面的例子之后，不知道对Bit-Map是否有了一个感性的认识。下面说下Bit-Map的建立过程。

**1****）开辟定长数组**

Bit-Map会声明一个定长的byte/int数组，之后将数组内元素的所有Bit位均置为0，如下图：

[![image](/images/2012/03/image_thumb2.png "image")](/images/2012/03/image2.png) 

**2****）遍历数据，并插入Bit-Map**

上例来说，就会遍历array{1, 2, 5, 8, 10}，并将所有的元素均插入Bit-Map中。Bit-Map是Hash的极致，那么key即为array[i]/8，value即在byte中的位置array[i]%8。而实际中为了效率，hash函数可能会有些出入。如下：   <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="568">           <p>Byte: MASK = 0X07; SHIFT = 3; Integer: MASK = 0X1F; SHIFT = 5; 

// i&gt;&gt; SHIFT =&gt; i/8; i &amp; MASK =&gt; i%8

set(i): array[i&gt;&gt;SHITF] |= (1 &lt;&lt; (i &amp; MASK);

// return 0: not exist

isExist(i) : return array[i&gt;&gt;SHIFT] &amp; (1 &lt;&lt; (i &amp; MASK);
         </td>       </tr>     </tbody></table> </p>  

遍历插入之后的数据应该是这样的：

[![image](/images/2012/03/image_thumb3.png "image")](/images/2012/03/image3.png) 

**3****、Bit-Map应用**

建立了Bit-Map之后，就可以方便的使用了。一般来说Bit-Map可作为数据的查找、去重、排序等操作。

如上面提及的10,000,000个数据存储问题，用Integer存储，耗费4G内存。改成Bit-Map，耗费125MB内存。但是实际中，可能由于数据中最大最小值相差太大，如{1,2 99999}，只有三个数，但是最大最小相差悬殊，该方法就不适用了。

查找和去重都好理解，至于排序，有点类似桶排序，每个byte都是一个桶。至于应用实例，自个用的少，copy别人的吧。

**1)****已知某个文件内包含一些电话号码，每个号码为8位数字，统计不同号码的个数。**

8位最多99 999 999，大概需要99m个bit，大概10几m字节的内存即可。可以理解为从0-99 999 999的数字，每个数字对应一个Bit位，所以只需要99M个Bit==1.2MBytes，这样，就用了小小的1.2M左右的内存表示了所有的8位数的电话。

**2)2.5****亿个整数中找出不重复的整数的个数，内存空间不足以容纳这2.5亿个整数。**

将bit-map扩展一下，用2bit表示一个数即可：0表示未出现；1表示出现一次；2表示出现2次及以上，即重复，在遍历这些数的时候，如果对应位置的值是0，则将其置为1；如果是1，将其置为2；如果是2，则保持不变。或者我们不用2bit来进行表示，我们用两个bit-map即可模拟实现这个2bit-map，都是一样的道理。
  > 参考资料：
> 
> 码农：[海量数据处理专题（四）——Bit-map](http://blog.redfox66.com/post/2010/09/26/mass-data-4-bitmap.aspx)
