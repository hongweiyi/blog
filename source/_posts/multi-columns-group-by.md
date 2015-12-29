title: 多列group-by及其应用场景
tags:
  - group by
  - SQL
id: 838
categories:
  - 技术分享
date: 2013-05-28 23:31:56
---

**1、前言**

一直以来都对多列group by没什么感觉，也不知道它到底要干嘛。这篇文章就粗略的讲一下什么是多列groupby，以及它的应用场景简例，没有什么深度，仅作为自己的知识笔记。
  > **BTW: **没感觉应该是因为一直没怎么接触数据库吧，罪过罪过(- -)<!--more-->  

**2、group by**

group by是SQL的语法，它一般结合一些聚合函数(aggregate functions)一起使用，如：SUM、AVG、MAX等。单列group by，比如group by columnA，就意味着，将A列中，所有相同的值放在同一个group。如下：
 <center>   <table border="1" cellspacing="0" cellpadding="0"><tbody>       <tr>         <td valign="top" width="119">           

**purchase_date**
         </td>          <td valign="top" width="131">           

**item**
         </td>          <td valign="top" width="95">           

**items_purchased**
         </td>       </tr>        <tr>         <td valign="top" width="119">           

2013-03-25
         </td>          <td valign="top" width="131">           

Wireless Mouse
         </td>          <td valign="top" width="95">           

2
         </td>       </tr>        <tr>         <td valign="top" width="119">           

2013-03-25
         </td>          <td valign="top" width="131">           

Wireless Mouse
         </td>          <td valign="top" width="95">           

5
         </td>       </tr>        <tr>         <td valign="top" width="119">           

2013-03-25 
         </td>          <td valign="top" width="131">           

MacBook Pro
         </td>          <td valign="top" width="95">           

1
         </td>       </tr>        <tr>         <td valign="top" width="119">           

2013-04-01
         </td>          <td valign="top" width="131">           

Paper Clips
         </td>          <td valign="top" width="95">           

20
         </td>       </tr>        <tr>         <td valign="top" width="119">           

2013-04-01 
         </td>          <td valign="top" width="131">           

Stapler
         </td>          <td valign="top" width="95">           

3
         </td>       </tr>        <tr>         <td valign="top" width="119">           

2013-04-01
         </td>          <td valign="top" width="131">           

Paper Clips
         </td>          <td valign="top" width="95">           

15
         </td>       </tr>        <tr>         <td valign="top" width="119">           

2013-05-15 
         </td>          <td valign="top" width="131">           

DVD player
         </td>          <td valign="top" width="95">           

3
         </td>       </tr>        <tr>         <td valign="top" width="119">           

2013-05-15
         </td>          <td valign="top" width="131">           

DVD player
         </td>          <td valign="top" width="95">           

8
         </td>       </tr>        <tr>         <td valign="top" width="119">           

2013-05-15
         </td>          <td valign="top" width="131">           

Stapler
         </td>          <td valign="top" width="95">           

5
         </td>       </tr>        <tr>         <td valign="top" width="119">           

2013-05-16
         </td>          <td valign="top" width="131">           

MacBook Pro
         </td>          <td valign="top" width="95">           

2
         </td>       </tr>     </tbody></table> </center>  

用户需要知道，哪天购买的东西最多，那就可以用group by，SQL如下：
  > select purchase_date, count(items_purchased) as purchased_count 
> 
> from Purchases group by purchase_date order by purchased_count desc; <center>   <table border="1" cellspacing="0" cellpadding="0" width="247"><tbody>       <tr>         <td valign="top" width="129">           

**purchase_date**
         </td>          <td valign="top" width="116">           

**purchased_count**
         </td>       </tr>        <tr>         <td valign="top" width="129">           

2013-04-01
         </td>          <td valign="top" width="116">           

38
         </td>       </tr>        <tr>         <td valign="top" width="129">           

2013-05-15 
         </td>          <td valign="top" width="116">           

16
         </td>       </tr>        <tr>         <td valign="top" width="129">           

2013-03-25
         </td>          <td valign="top" width="116">           

8
         </td>       </tr>        <tr>         <td valign="top" width="129">           

2013-05-16
         </td>          <td valign="top" width="116">           

2
         </td>       </tr>     </tbody></table> </center>  

**3、多列group by**

同上，用户如果想知道，给定日期，该天商品购买的详细情况，这就需要使用多列group by了。

多列group by，比如group by columnA, columnB，就意味着，将A、B两列中均相同的值放在同一个group。SQL可以这样写：
  > <pre>select purchase_date, item, sum(items_purchased) as > 
> &quot;total items&quot; from Purchases group by item, purchase_date;</pre>

返回结果如下：

<center>
  <table border="1" cellspacing="0" cellpadding="0" width="347"><tbody>
      <tr>
        <td valign="top" width="115">

**purchase_date**

        </td>

        <td valign="top" width="123">

**item**

        </td>

        <td valign="top" width="107">

**Total Items**

        </td>
      </tr>

      <tr>
        <td valign="top" width="115">

2013-03-25

        </td>

        <td valign="top" width="123">

Wireless Mouse

        </td>

        <td valign="top" width="107">

7

        </td>
      </tr>

      <tr>
        <td valign="top" width="115">

2013-03-25

        </td>

        <td valign="top" width="123">

MacBook Pro

        </td>

        <td valign="top" width="107">

1

        </td>
      </tr>

      <tr>
        <td valign="top" width="115">

2013-04-01

        </td>

        <td valign="top" width="123">

Paper Clips

        </td>

        <td valign="top" width="107">

35

        </td>
      </tr>

      <tr>
        <td valign="top" width="115">

2013-04-01

        </td>

        <td valign="top" width="123">

Stapler

        </td>

        <td valign="top" width="107">

3

        </td>
      </tr>

      <tr>
        <td valign="top" width="115">

2013-05-15

        </td>

        <td valign="top" width="123">

DVD player

        </td>

        <td valign="top" width="107">

11

        </td>
      </tr>

      <tr>
        <td valign="top" width="115">

2013-05-15

        </td>

        <td valign="top" width="123">

Stapler

        </td>

        <td valign="top" width="107">

5

        </td>
      </tr>

      <tr>
        <td valign="top" width="115">

2013-05-16

        </td>

        <td valign="top" width="123">

MacBook Pro

        </td>

        <td valign="top" width="107">

2

        </td>
      </tr>
    </tbody></table>
</center>

多列group by会将被groupby的多列合成相同的group，并可对每个group进行相应的聚合，如前面的sum()。

&#160;

> **参考文献：**
> 
> **[ProgrammerInterview.com](http://t.cn/zHJvZ7E)**
> 
> [StackOverflow](http://stackoverflow.com/questions/2421388/using-group-by-on-multiple-columns) - Using group by on multiple columns

> **BTW:** 技术还是看英文吧。
