title: Solr Query Parser学习有感
tags:
  - Solr
id: 845
categories:
  - 技术分享
date: 2013-08-31 10:55:12
---

**1、前言**

	早上了解QueryParser的时候，发现了一个有意思的插件-SwitchQueryParser。看名字有种高端大气上档次的感觉，但是用起来好像就没这种感觉了，因为好像功能确实有限，这插件只是提供了一种更为&ldquo;友好&rdquo;的filter query功能。

<!--more-->

	&nbsp;

	**2、细说**

	插件的语法如下：

> fq={!switch case.foo=XXX case.bar=zzz case.yak=qqq}foo

	这个语法是官方文档里面的，这算个什么嘛，还真只是语法，一点逻辑都没有！！！我就写了带一点点逻辑的查询语句如下：

> fq={!switch case.cs=city:changsha case.bj=city:beijing}cs

	这句话翻译过来，其实就是：

> fq=city:changsha

	因为在SwitchQParserPlugin的parse()中，只是找到了localParams的v属性（即cs），再从SWITCH_CASE对应的字段，获得相应的查询语句后，返回子查询对象。由于Solr模块可移植性强加上逻辑实在是过于简单，所以做低版本移植也非常方便，几(M&eacute;i)分(yǒu)钟(y&agrave;n)搞(zh&egrave;ng)定(ā)。

	如下：

	[![image](http://hongweiyi.com/wp-content/uploads/2013/08/image_thumb.png "image")](http://hongweiyi.com/wp-content/uploads/2013/08/image.png)&nbsp;

	部分源码：

[code lang="java"]
/** SwitchQParserPlugin */
@Override
public Query parse() throws SyntaxError {
  String val = localParams.get(QueryParsing.V);

  // we don't want to wrapDefaults arround params, because then 
  // clients could add their own switch options 
  String subQ = localParams.get(SWITCH_DEFAULT);
  subQ = StringUtils.isBlank(val)
    ? localParams.get(SWITCH_CASE, subQ)
    : localParams.get(SWITCH_CASE + &quot;.&quot; + val.trim(), subQ);

  if (null == subQ) {
    throw new SyntaxError(&quot;No &quot;+SWITCH_DEFAULT+&quot;, and no switch case matching specified query string: \&quot;&quot; + val + &quot;\&quot;&quot;);
  }

  subParser = subQuery(subQ, null);
  return subParser.getQuery();
}
[/code]

	&nbsp;

	**3、有感**

	这个插件的使用场景我觉得更多的是在业务层使用，由业务层封装多个case，将多个case的值暴露给更上层的业务，提供一个较为友好的交互方式。比如下面这种（肯定实际不是这么做的）：

	[![QQ图片20130831103315](http://hongweiyi.com/wp-content/uploads/2013/08/QQ20130831103315_thumb.jpg "QQ图片20130831103315")](http://hongweiyi.com/wp-content/uploads/2013/08/QQ20130831103315.jpg)

	总的来说，这个插件对于我们现在的系统而言，用处不大，但是Solr的这种查询方式，这种插件式编码方式，很值得我们借鉴和学习。
