title: 人生苦短，我用Python
tags:
  - Python
id: 243
categories:
  - 技术分享
date: 2012-01-20 16:40:51
---

用脚本语言的原因就是因为其语法简单，使用方便，但是好像一直没有把握其中的精华。虽然仔细读过网上转载过N次的[Python少打字小技巧](http://www.joynb.net/blog/archives/120)，但还是无法很好的运用。

 <!--more-->  

今天逛论坛的时候，看到了一个网友发了一个[ip地址排序](http://bbs.chinaunix.net/thread-823862-1-1.html)的帖子，代码行数60行，核心代码30多行。没有仔细读，看看评论的时候，看到了另一个解法，总代码6行，核心代码4行。用了列表解析、匿名函数等特性，还有内置函数，代码如下：

``` python
iplist=[‘192.168.1.33′,’10.5.1.3′,’10.5.2.4′,’202.98.96.68′,’13.12.1.1’]  
def ip2int(s):  
    l = [int(i) for i in s.split(‘.’)]  
    return (l[0] << 24) | (l[1] << 16) | (l[2] << 8 ) | l[3]  
      
iplist.sort(lambda x, y: cmp(ip2int(x), ip2int(y)))  
print iplist 
```

按照[Python少打字小技巧](http://www.joynb.net/blog/archives/120)里的例子，这个代码核心部分可以缩减到2行（其实可以只有1行），不过，可读性就不敢恭维咯。


``` python
iplist=[‘192.168.1.33′,’10.5.1.3′,’10.5.2.4′,’202.98.96.68′,’13.12.1.1’]  
p = lambda ip: sum( [ int(k)*v for k, v in zip(ip.split(‘.’), [1<<24, 65536, 256, 1])]);  
iplist.sort(lambda x, y: cmp(p(x), p(y)))  
print iplist 
```

既然说到短小精悍，就写写Python的一些特性和函数吧：lambda、map、reduce、filter

**lambda**

匿名函数，和c/c++中的宏定义类似，格式为：lambda [参数] : [表达式]，如：lambda x : x+2，调用的时候可以用一个变量接收这个函数，并传入参数。

``` python
lam = lambda x : x+2  
print lam(2)  
print (lambda x : x+2)(2) 
```

有些人觉得lambda也就这点功能，但笔者认为，这才是脚本语言，点题：人生苦短，我用Python！这位[**博主**](http://www.cnblogs.com/coderzh/archive/2010/04/30/python-cookbook-lambda.html)写得不错，可以参考参考。

**map，reduce，filter**

其实lambda很多地方都是用在上面这三个函数上面，map(func,seq)、reduce(func,seq,initial=None)、filter(bool_func,seq)。

* map：对seq中的每一个对象都执行func函数，并返回一个列表；
* reduce：对seq中的每两个对象依次执行func函数，并返回一个值，如1+2+3+4；
* filter：过滤掉seq中不符合bool_func的对象。

虽然这三个函数中func都可以使用def来定义，但是如果程序需要的逻辑比较简单的话，用lambda就足够了。

``` python
ls = [1,2,3,4,5]  
# every item in ls plus 2  
print map(lambda x : x+2, ls)  
# sum(ls)  
print reduce(lambda x, y : x+y, ls)  
# filter the item which less than 2  
print filter(lambda x : x > 2 and True or False, ls)  
```
