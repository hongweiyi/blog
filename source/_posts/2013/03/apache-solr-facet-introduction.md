title: Apache Solr —— Facet介绍
date: 2013-03-23 14:31:00
categories: 技术分享
tags: Solr
---

### 1、什么是Faceted Search

`Facet['fæsɪt]`很难翻译，只能靠例子来理解了。Solr作者Yonik Seeley也给出更为直接的名字：导航（Guided Navigation）、参数化查询（Paramatic Search）。

<!--more-->

<center><div style="width: 80%;">![facet-1](/images/facet-1.png)</div></center>

上面是比较直接的Faceted Search例子，品牌、产品特征、卖家，均是Facet。而Apple、Lenovo等品牌，就是Facet values或者说Constraints，而Facet values所带的统计值就是Facet count/Constraint count。

### 2、Facet使用
> q = 超级本
facet = true
facet.field = 产品特性
facet.field = 品牌
facet.field = 卖家
> `http://.../select?q=超级本&facet=true&wt=json&facet.field=品牌&facet.field=产品特性&facet.field=卖家`

```
"facet_counts": {
"facet_fields": {
  "品牌": [
    "Apple", 4,
    "Lenovo", 39
      …]
  "产品特性": [
    "显卡", 42,
    "酷睿", 38
      …]

  …}}
```

也可以提交查询条件，设置fq(filter query)。

> q = 电脑
facet = true
fq = 价格:[8000 TO \*]
facet.mincount = 1 // fq将不符合的字段过滤后，会显示count为0
facet.field = 产品特性
facet.field = 品牌
facet.field = 卖家</blockquote>
> `http://.../select?q=超级本&facet=true&wt=json&fq=价格:[8000 TO *]&facet.mincount=1&facet.field=品牌&facet.field=产品特性&facet.field=卖家

``` json
"facet_counts": {
"facet_fields": {
  "品牌": [
    "Apple", 4,
    "Lenovo", 10
      …]
  "产品特性": [
    "显卡", 11,
    "酷睿", 20
      …]

  …}}
```

如果用户选择了Apple这个分类，查询条件中需要添加另外一个fq查询条件，并移除Apple所在的facet.field。

> `http://.../select?q=超级本&facet=true&wt=json&fq=价格:[8000 TO *]&fq=品牌:Apple&facet.mincount=1 &facet.field=产品特性&facet.field=卖家`

### 3、Facet参数

* **facet.prefix**: 限制constaints的前缀
* **facet.mincount=0**: 限制constants count的最小返回值，默认为0
* **facet.sort=count**: 排序的方式，根据count或者index
* **facet.offset=0**: 表示在当前排序情况下的偏移，可以做分页
* **facet.limit=100**: constraints返回的数目
* **facet.missing=false**: 是否返回没有值的field
* **facet.date**: Deprecated, use facet.range
* **facet.query**: 指定一个查询字符串作为Facet Constraint

```
facet.query = rank:[* TO 20]
facet.query = rank:[21 TO *]
```

``` xml
  <result numFound="27" ... />
  ...
  <lst name="facet_counts">
  <lst name="facet_queries">
    <int name="rank:[* TO 20]">2</int>
    <int name="rank:[21 TO *]">15</int>
  </lst>
 ...
```

* **facet.range:

`http://.../select?&facet=true&facet.range=price&facet.range.start=5000&facet.range.end=8000&facet.range.gap=1000`

``` json
 "facet_counts":{
  "facet_ranges":{
    "price":{
      "counts”:[
        "5000.0”,5,
        "6000.0”,2,
        "7000.0”,3,],
      "gap":1000.0,
      "start":5000.0,
      "end":8000.0}}}}
```

> **WARNING:**
>
> range范围是左闭右开，`[start, end)`

* **facet.pivot**

这个是Solr 4.0的新特性，pivot和facet一样难理解，还是用例子来讲吧。

> **Syntax:** facet.pivot=field1,field2,field3...
> **e.g.:** facet.pivot=comment_user, grade

|   |#docs|#docs grade:好|#docs 等级:中|#docs 等级:差|
|---|----|--------------|------------|-----------|
|comment_user:1|10|8|1|1|
|comment_user:2|20|18|2|0|
|comment_user:3|15|12|2|1|
|comment_user:4|18|15|2|1|

```
  "facet_counts":{
  "facet_pivot":{
   "comment_user, grade ":[{
     "field":"comment_user",
     "value":"1",
     "count":10,
     "pivot":[{
       "field":"grade",
       "value":"好",
       "count":8}, {
       "field":"grade",
       "value":"中",
       "count":1}, {
       "field":"grade",
       "value":"差",
       "count":1}]
     }, {
       "field":" comment_user ",
       "value":"2",
       "count":20,
       "pivot":[{
       ...
```

没有pivot机制的话，要做到上面那点可能需要多次查询：
> `http://...q=comment&fq=grade:好&facet=true&facet.field=comment_user`
> `http://...q=comment&fq=grade:中&facet=true&facet.field=comment_user`
> `http://...q=comment&fq=grade:差&facet=true&facet.field=comment_user`

> Facet.pivot - Computes a Matrix of Constraint Counts across multiple Facet Fields. by Yonik Seeley.
上面那个解释很不错，只能理解不能翻译。

> 参考资料：
> * [The Many Facets of Apache Solr](http://2011.lucene-eurocon.org/attachments/0002/8835/Seeley_Eurocon_SolrFacets_1_.pdf)
> * [SimpleFacetParameters Wiki](http://wiki.apache.org/solr/SimpleFacetParameters)
