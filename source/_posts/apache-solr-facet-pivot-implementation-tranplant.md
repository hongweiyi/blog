title: Apache Solr —— Facet Pivot实现与低版本移植
date: 2013-03-27 23:12:00
categories: 技术分享
tags: Solr
---

### 1、前言

Solr升级到4.0后便有了一个新功能，就是facet.pivot，关于pivot的介绍可以看上一篇文章：[《Apache Solr – Facet介绍》](http://hongweiyi.com/2013/03/apache-solr-facet-introduction/) 这篇主要讲述Pivot的内部实现机制，以及如何将这个功能移植到低版本的Solr中来。

<!--more-->

### 2、Pivot实现机制

> `http://...?q=*:*&facet=true&&facet.field=province&facet.field=city&facet.pivot=province,city&wt=json`

#### 2.1 获得所有pivots参数列表

> e.g.: ["province,city", …]

#### 2.2 将pivots交给PivotFacetHelper处理

``` java
// FacetComponent.java -> process
// e.g.: ["province,city"]
String[] pivots = params.getParams(FacetParams.FACET_PIVOT);
if (pivots != null && pivots.length > 0) {
	PivotFacetHelper pivotHelper = new PivotFacetHelper(rb.req,
						rb.getResults().docSet, params, rb);
	// process each pivots individually
	NamedList v = pivotHelper.process(pivots);
	if (v != null) {
		counts.add(PIVOT_KEY, v);
	}
}
```

#### 2.3 解析pivot，获得每个field –> pivot: field subField fnames

``` Java
// PivotFacetHelper.java -> process
for (String pivot : pivots) {
	// …

	String[] fields = pivot.split(",");

	if (fields.length < 2) {
	    throw new SolrException(ErrorCode.BAD_REQUEST,
			"Pivot Facet needs at least two fields: " + pivot);
	}

	String field = fields[0];
	String subField = fields[1];

	// the rest of fields
	Deque<String> fnames = new LinkedList<String>();
	for (int i = fields.length - 1; i > 1; i--) {
		fnames.push(fields[i]);
	}
	// …
}
```

#### 2.4 获得第一级field的facets

``` java
// PivotFacetHelper.java -> process
// e.g.: field: province
//        superFacets : ["Jiangsu", 4, "Hunan", 3, "Guangdong", 2, "Beijing", 1, "Zhejiang", 1]
NamedList<Integer> superFacets = this.getTermCounts(field);
```

#### 2.5 递归调用pivotResult = doPivot(superFacets, field, subField, fnames, docs)

``` java
        // …
        // super.key usually == pivot unless local-param 'key' used
        pivotResponse.add(key,
        doPivots(superFacets, field, subField, fnames, base));
	}
	return pivotResponse;
}

// PivotFacetHelper.java -> doPivots
protected List<NamedList<Object>> doPivots(NamedList<Integer> superFacets, String field, String subField, Deque<String> fnames, DocSet docs) throws IOException {

	// …
	// 遍历该级所有facet，获得该facet下所有下一级facet数据…
    // ["Jiangsu", "Hunan", "Guangdong", "Beijing", "Zhejiang"]
	for (Map.Entry<String, Integer> kv : superFacets) {
		// Only sub-facet if parent facet has positive count - still may not
		// be any values for the sub-field though
		if (kv.getValue() >= minMatch) {

			// …
			// pivot就是该级facet，添加它相关数据（field、value、count）
			SimpleOrderedMap<Object> pivot = new SimpleOrderedMap<Object>();
			pivot.add("field", field);
			if (null == fieldValue) {
				pivot.add("value", null);
			} else {
			    // termval：当前facets、field下，所包含的值
                // e.g.: “Jiangsu”
				termval = new BytesRef();
				ftype.readableToIndexed(fieldValue, termval);
				pivot.add("value", ftype.toObject(sfield, termval));
			}
			pivot.add("count", kv.getValue());

			// subField为空，这一级facet的这个pivot递归遍历完毕
			if (subField == null) {
				values.add(pivot);
			// subField不为空，继续递归遍历
			} else {
			    // subset: 当前facet下，包含field:value的文档子集
			    // field:value -> “province”:“Jiangsu”
				DocSet subset = null;

				if (null == termval) {
					DocSet hasVal = searcher.getDocSet(new TermRangeQuery(
							field, null, null, false, false));
					subset = docs.andNot(hasVal);
				} else {
					String term = new String(new String(termval.bytes,
							termval.offset, termval.length));

					Query query = new TermQuery(new Term(field, term));
					subset = searcher.getDocSet(query, docs);
				}
				super.docs = subset;// used by getTermCounts()

                 // nl: 获得当前docs下，下一级field的facets。
                 // nl我理解为next level
                 // e.g.: subField: “city”
                 //         nl: [“Suzhou”, 3, “Nanjing”, 1]
				NamedList<Integer> nl = this.getTermCounts(subField);
				if (nl.size() >= minMatch) {
                     // 继续下一层迭代
					pivot.add(	"pivot",doPivots(nl, subField, nextField, fnames, subset));
					values.add(pivot); // only add response if there are
											// some counts
				}
			}
		}
	}

		// put the field back on the list
	fnames.push(nextField);
	return values;
}
```

> 新版添加新机制方式可以参考，并不是在源码基础上大刀阔斧的改。而是添加了一个新类，只需原有获取facet流程逻辑上，添加少数几行代码即可，这也算是模块化思想么？

### 3、移植Pivot功能到低级版本

> 这里是从solr4.0移植到solr3.5

移植步骤倒也还方便，主要是添加PivotFacetHelper类，剩下的就是一步一步修复红叉叉了。

``` java
Index: core/src/java/org/apache/solr/handler/component/FacetComponent.java
===================================================================
--- core/src/java/org/apache/solr/handler/component/FacetComponent.java	(Version unknown old)
+++ core/src/java/org/apache/solr/handler/component/FacetComponent.java	(Version unknown new)

46a47,48
> 	static final String PIVOT_KEY = "facet_pivot";
>
66c68,79
<
---
> 			NamedList<Object> counts = f.getFacetCounts();
> 			// e.g.: ["province,city"]
> 			String[] pivots = params.getParams(FacetParams.FACET_PIVOT);
> 			if (pivots != null && pivots.length > 0) {
> 				PivotFacetHelper pivotHelper = new PivotFacetHelper(rb.req,
> 						rb.getResults().docSet, params, rb);
> 				// process each pivots individually
> 				NamedList v = pivotHelper.process(pivots);
> 				if (v != null) {
> 					counts.add(PIVOT_KEY, v);
> 				}
> 			}
68c81
< 			rb.rsp.add("facet_counts", f.getFacetCounts());
---
> 			rb.rsp.add("facet_counts", counts);
```

``` java
Index: solrj/src/java/org/apache/solr/common/params/FacetParams.java
===================================================================
--- solrj/src/java/org/apache/solr/common/params/FacetParams.java	(Version unknown old)
+++ solrj/src/java/org/apache/solr/common/params/FacetParams.java	(Version unknown new)

92a93,98
> 	/**
> 	 * Comma separated list of fields to pivot
> 	 *
> 	 * example: author,type (for types by author / types within author)
> 	 */
> 	public static final String FACET_PIVOT = FACET + ".pivot";
94a101,106
> 	 * Minimum number of docs that need to match to be included in the sublist
> 	 *
> 	 * default value is 1
> 	 */
> 	public static final String FACET_PIVOT_MINCOUNT = FACET_PIVOT + ".mincount";
> 	/**
```

``` java
Index: core/src/java/org/apache/solr/request/SimpleFacets.java
===================================================================
--- core/src/java/org/apache/solr/request/SimpleFacets.java	(Version unknown old)
+++ core/src/java/org/apache/solr/request/SimpleFacets.java	(Version unknown new)

74,76c74,77
< 	String facetValue; // the field to or query to facet on (minus local params)
< 	DocSet base; // the base docset for this particular facet
< 	String key; // what name should the results be stored under
---
> 	protected String facetValue; // the field to or query to facet on (minus
> 									// local params)
> 	protected DocSet base; // the base docset for this particular facet
> 	protected String key; // what name should the results be stored under
92,93c93,94
< 	void parseParams(String type, String param) throws ParseException,
< 			IOException {
---
> 	protected void parseParams(String type, String param)
> 			throws ParseException, IOException {
```

```
Index: core/src/java/org/apache/solr/schema/FieldType.java
===================================================================
--- core/src/java/org/apache/solr/schema/FieldType.java	(Version unknown old)
+++ core/src/java/org/apache/solr/schema/FieldType.java	(Version unknown new)
394a399,411
> 	public Object toObject(SchemaField sf, BytesRef term) {
> 		final CharsRef ref = new CharsRef(term.length);
> 		indexedToReadable(term, ref);
> 		final Fieldable f =  createField(sf, ref.toString(), 1.0f);
> 		return toObject(f);
> 	}
>
> 	/** Given an indexed term, append the human readable representation */
> 	public CharsRef indexedToReadable(BytesRef input, CharsRef output) {
> 		UnicodeUtil.UTF8toUTF16(input, output);
> 		return output;
> 	}
>
418a436,441
> 	/** Given the readable value, return the term value that will match it. */
> 	public void readableToIndexed(CharSequence val, BytesRef result) {
> 		final String internal = readableToIndexed(val.toString());
> 		UnicodeUtil.UTF16toUTF8(internal, 0, internal.length(), result);
> 	}
>
```

```
Index: core/src/java/org/apache/solr/handler/component/PivotFacetHelper.java
===================================================================
--- core/src/java/org/apache/solr/handler/component/PivotFacetHelper.java	(Version unknown old)
+++ core/src/java/org/apache/solr/handler/component/PivotFacetHelper.java	(Version unknown new)

144c152,155
<        Query query = new TermQuery(new Term(field, termval));
---
>        String term = new String(new String(termval.bytes,
>                       termval.offset, termval.length));
>
>        Query query = new TermQuery(new Term(field, term));
147,148c158,159
<        super.docs = subset;// used by getTermCounts()
<
---
>        super.base = subset;// used by getTermCounts()
>
```

> 没弄过patch，上面的就将就着看吧。

> **WARNING**:
>
> 3.x中，SimpleFacet组件中有两个成员，docs/base。在4.x中，这两个成员改名为docsOrig/docs。名字倒是清晰了，就是移植的时候没注意倒是挺麻烦的。

> **WARNING**:
>
> BytesRef这个类，在4.x中改动比较大，而且Term也原生支持BytesRef。为了偷懒，就直接将BytesRef.bytes转换成了String。

> **WARNING**:
>
>  上面还有一些小类，没有写上，有红叉叉自己改改应该问题不大。</blockquote>
