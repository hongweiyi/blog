title: Apache Solr —— DistributedSearch
date: 2013-04-08 18:04:00
categories: 技术分享
tags: Solr
---

### 1、前言

当索引太大以致单台服务器的磁盘无法承受了，当一个简单的查询实在要耗费过多的时间，可以考虑使用Solr的分布式索引机制，或者配置一台多核机制（multicore）。当Solr配置了这样的机制，实质上就是将大索引分成了多个小索引分布在了不同服务器上，或者将请求发到多核，充分利用服务器CPU资源。

Solr会将请求请求分发不同shards（理解为地址）上，并合并所有请求结果并返回给客户端。那么，这个分布式查询内部是怎么实现的呢？

<!--more-->

### 2、机制解析

一个SearchComponent如果作为Distributed SearchComponent，需要重写以下四个方法：

* distributedProcess()
* modifyRequest()
* handleResponses()
* finishStage()

同时，Distributed Search主要有4个主要阶段：

* **Start**(ResponseBuilder.STAGE_START)
* **Query Parse**(ResponseBuilder.STAGE_PARSE_QUERY)
* **Execute Query**(ResponseBuilder.STAGE_EXECUTE_QUERY)
* **Get Fields**(ResponseBuilder.STAGE_GET_FIELDS)</blockquote>
* **Done**(ResponseBuilder.STAGE_DONE)。

在SearchHandler中，基本的**分布式查询算法**如下：

#### 1. 如果不在STAGE_DONE的阶段，循环以下流程；
#### 2. 发起分布式处理的组件会不断查询是否需要进行分布式处理。如果有的话（distributedProcess），返回4个阶段中一个状态并创建一个 ShardRequest并将它添加到一个队列（rb.outgoing)中；

> Components可以指定一些purpose字段，可以定制一些特殊的请求处理。如下：

``` java
public void handleResponses(ResponseBuilder rb, ShardRequest sreq) {
  if (!rb.doFacets )
     return;

  if ((sreq.purpose & ShardRequest.PURPOSE_GET_FACETS) != 0) {
     countFacets(rb, sreq);
  } else if ((sreq.purpose & ShardRequest.PURPOSE_REFINE_FACETS ) != 0) {
     refineFacets(rb, sreq);
  }
}
```

> modifyRequest()是用来精简shard请求的，该方法都是加在了ResponseBuilder.addRequest()中，实例可以参考FacetComponent.modifyRequest。

``` java
public void addRequest(SearchComponent me, ShardRequest sreq) {
  outgoing.add(sreq);
  if ((sreq.purpose & ShardRequest.PURPOSE_PRIVATE)==0) {
   // if this isn't a private request, let other components modify it.
   for (SearchComponent component : components) {
      if (component != me) {
        component.modifyRequest( this, me, sreq);
      }
    }
  }
}
```

#### 3. 取出队列（rb.outgoing)中所有 ShardRequest，并根据其配置将请求发到相应的 Shards；
#### 4. 如果队列为空，则等待接收到响应，并处理（handleResponse)；

> 你可以合并文档id神马的。QueryComponent.handleResponse

#### 5. 进入下一轮循环之前，先调用finishStage()

> 进行该轮的收尾工作，比如说将为null的doc从responseDocs中移除。

``` java
// SearchHandler.handleRequestBody()
if (rb.shards == null) {
  // a normal non-distributed request
  // ...
} else {
  // a distributed request

  HttpCommComponent comm = new HttpCommComponent();

  if (rb.outgoing == null ) {
    rb.outgoing = new LinkedList<ShardRequest>();
  }
  rb.finished = new ArrayList<ShardRequest>();

  int nextStage = 0;
  do {
    rb.stage = nextStage;
    nextStage = ResponseBuilder.STAGE_DONE;

    // call all components
    for( SearchComponent c : components ) {
      // the next stage is the minimum of what all components report
      nextStage = Math.min(nextStage, c.distributedProcess(rb));
    }


    // check the outgoing queue and send requests
    while (rb.outgoing.size() > 0) {

      // submit all current request tasks at once
      while (rb.outgoing.size() > 0) {
        ShardRequest sreq = rb.outgoing.remove(0);
        sreq.actualShards = sreq.shards ;
        if (sreq.actualShards ==ShardRequest.ALL_SHARDS) {
          sreq.actualShards = rb.shards ;
        }
        sreq.responses = new ArrayList<ShardResponse>();

        // TODO: map from shard to address[]
        for (String shard : sreq.actualShards ) {
          ModifiableSolrParams params = new ModifiableSolrParams(sreq.params );
          params.remove(ShardParams.SHARDS);      // not a top-level request
          params.remove( "indent");
          params.remove(CommonParams.HEADER_ECHO_PARAMS);
          params.set(ShardParams.IS_SHARD, true );  // a sub (shard) request
          String shardHandler = req.getParams().get(ShardParams.SHARDS_QT );
          if (shardHandler == null) {
            params.remove(CommonParams.QT);
          } else {
            params.set(CommonParams.QT, shardHandler);
          }
          // You can see CommonsHttpSolrServer.request.
          comm.submit(sreq, shard, params);
        }
      }


      // now wait for replies, but if anyone puts more requests on
      // the outgoing queue, send them out immediately (by exiting
      // this loop)
      while (rb.outgoing.size() == 0) {
        ShardResponse srsp = comm.takeCompletedOrError();
        if (srsp == null) break;  // no more requests to wait for

        // Was there an exception?  If so, abort everything and
        // rethrow
        if (srsp.getException() != null) {
          comm.cancelAll();
          if (srsp.getException() instanceof SolrException) {
            throw (SolrException)srsp.getException();
          } else {
            throw new SolrException(SolrException.ErrorCode.SERVER_ERROR, srsp.getException());
          }
        }

        rb.finished.add(srsp.getShardRequest());

        // let the components see the responses to the request
        for(SearchComponent c : components ) {
          c.handleResponses(rb, srsp.getShardRequest());
        }
      }
    }

    for(SearchComponent c : components) {
        c.finishStage(rb);
     }

    // we are done when the next stage is MAX_VALUE
    // BTW ResponseBuilder.STAGE_DONE == Integer.MAX_VALUE
  } while (nextStage != Integer.MAX_VALUE);
}
```

### 3、其他

有意思的是，我从Solr 3.5的reference中看到一句这样的话：

> It is up to you to get all your documents indexed on each shard of your server farm. Solr does not include out-of-the-box support for distributed indexing, but your method can be as simple as a round robin technique. Just index each document to the next server in the circle.

> Solr本身并不自带分布式的一些机制，索引如何分块还是靠自己写逻辑（如：轮询、hash取模等）。难怪我测试multicore的时候，shards的地址参数还得自己写，这一阶段对于用户来说不是透明的，所谓的分布式查询，也就是每个查询都要访问多台服务器，之后将多台服务器的结果和合并起来，这样看来，对每台服务器的压力来说感觉都是一样的诶（虽然可以不同业务分响应的小集群）。

不知从哪个版本（好像是4.0）推出了Solr Cloud，官方这样介绍的：

> SolrCloud is the name of a set of new distributed capabilities in Solr. Passing parameters to enable these capabilities will enable you to set up a highly available, fault tolerant cluster of Solr servers. Use SolrCloud when you want high scale, fault tolerant, distributed indexing and search capabilities.

似乎这样才算是分布式解决方案，嗯，以后再深入看看Solr Cloud Architecture。

> 参考资料：
* [SolrCloud](http://wiki.apache.org/solr/SolrCloud)
* [DistributedSearchComponent](http://wiki.apache.org/solr/WritingDistributedSearchComponents)
