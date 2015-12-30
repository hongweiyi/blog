title: Apache Solr —— 常用数据结构（二）
date: 2013-04-08 18:04:00
categories: 技术分享
tags: [Solr, 数据结构]
---

### 1、前言

继上篇[《Apache Solr – 常用数据结构（一）》](http://hongweiyi.com/2013/04/apache-solr-data-structrue-part-1/)介绍了NamedList和DocSet之后，这篇主要介绍ResponseBuilder和Query。

<!--more-->

### 2、ResponseBuilder

每个组件（ QueryComponent、FacetComponent、MoreLikeThisComponent、HighlightComponent、 DebugComponent等)的操作都是围绕着 ResponseBuilder的实例来进行的，因为这个实例包含了全部的请求以及响应的信息。

ResponseBuilder是在SearchHandler的handleRequestBody方法中创建的，主要包含两个成员，SolrQueryRequest和SolrQueryResponse。还有一个components成员，用来装各种SearchComponent，但是好像这个成员并无太大的用武之地。推测作者设计ResponseBuilder类的原因应该是为了降低组件与组件之间的耦合度，让代码逻辑更为清晰。

各种Components的prepare阶段主要是丰满了rb对象，将request成员中各种参数一一取出，赋值给rb。并在process方法中将结果写入response成员。

如FacetComponent的prepare方法：

``` java
public void prepare(ResponseBuilder rb) throws IOException {
  if (rb.req .getParams().getBool(FacetParams.FACET, false)) {
    rb.setNeedDocSet( true);
    rb. doFacets = true ;
  }
}
```


### 3、Query

Lucene提供了多种多样的Query的实现，大多数都在org.apache.lucene.search包下面，这些Query结合起来可以实现非常复杂的查询。Query类有MatchAllDocsQuery、BooleanQuery、TermQuery等，系统通过QueryParser就会将查询解析为一个Query。如"*:*"就会解析成MatchAllDocsQuery，"queryStr1 OR queryStr2"就会解析成一个BooleanQuery等。

Query是大多数场景的开始，如果没有Query的话，系统就无法查询文档更别说进行打分（Score）。Query是Scorer的催化剂，并且负责创建和协调它，而Weight则提供了一个Query的中间形态，任何Searcher的依赖状态都必须存放在Weight对象中，而不是Query对象，所以大多数Query的实现类都需要提供一个Weight的实现。

Query的深入剖析好像是需要对Solr、Lucene有一个专家级的理解了，初学者的我就不多言了。

可参考文档：

* [Apache Lucene - Scoring](http://lucene.apache.org/core/3_6_2/scoring.html)
* [Package - org.apache.lucene.search - query](http://lucene.apache.org/core/3_6_2/api/core/org/apache/lucene/search/package-summary.html#query)
* [Lucene打分机制和Similarity模块](http://my.oschina.net/BreathL/blog/51498)

### 4、结尾

以上包括上一篇博文我都只能粗略的理解一些，并不能深入详细的理解及运用，需要在后期的学习及实践的过程中继续深入了。
