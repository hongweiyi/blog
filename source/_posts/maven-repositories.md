title: Maven笔记 - 仓库
tags:
  - java
  - Maven
id: 726
categories:
  - 技术分享
date: 2013-03-18 10:13:17
---

**1、前言**

	上一篇[博客](http://hongweiyi.com/2013/03/maven-coordinates-dependencies/)介绍了Maven坐标和依赖。在Maven中，任何一个依赖、插件或者项目构建的输出，都可以成为构件。而坐标和依赖是任何一个构件在Maven世界中的逻辑表示方式，构件的物理表示方式是文件，Maven则通过仓库（Repositories）来统一管理这些文件。Maven仓库是通过简单文件系统存储管理的，当遇到与仓库相关的问题，可以直接查找相关文件，方便定位问题。

<!--more-->

	&nbsp;

	**2、仓库的分类**

	对于Maven而言，仓库分为两类：本地仓库和远程仓库。当Maven根据坐标寻找构件的时候，它会首先查看本地仓库，如果本地仓库存在此构件，则直接使用；如果本地仓库不存在此构件，或者需要查看是否有更新的构件版本，Maven就会去远程仓库查找，发现需要的构件之后，下载到本地仓库再使用。如果本地仓库和远程仓库都有没有需要的构件Maven就会报错。

	这里有三个远程仓库：**中央仓库、私服和其他公共库**。中央仓库是Maven核心自带的远程仓库（http://repo1.maven.org/maven2），它包含了绝大部分的构件。私服是另一种特殊的远程仓库，为了节省带宽和时间，应该在局域网内架设一个私有的仓库服务器，用其代理所有外部的远程仓库。除了以上两种，还有很多公开的远程仓库，比如Java. Net Maven库（http://download.java.net/maven/2/）和JBoss Maven库（http://repository.jboss.com/maven2/）

> **IMPORTANT**：私服最重要的功能应该不在节省速度和带宽，而是可以部署内部构件，供企业内部开发人员使用。

	[![image](http://hongweiyi.com/wp-content/uploads/2013/03/image_thumb.png "image")](http://hongweiyi.com/wp-content/uploads/2013/03/image.png)一图胜千言

	&nbsp;

	**3、远程仓库的配置**

	这位弟兄将《Maven实战》这个部分都给Copy到网上了，并加以标注，可以直接参考：[http://t.cn/zYgRHb7](http://t.cn/zYgRHb7)

	&nbsp;

	**4、Maven镜像**

	如果仓库X可以提供仓库Y存储的所有内容，那么就可以认为X是Y的一个镜像。Maven中央仓库在国内经常不能正常访问，连接超时不能下载资源之类的。在公司内部还挺好，有私服可以用，但是回家之后那龟爬的网速就难说了，一般来说可以配置Maven镜像。配置需要修改~/.m2/settings.xml，内容如下：

[code lang="xml"]
&lt;settings&gt;
  &lt;!--…--&gt;
  &lt;mirrors&gt;
    &lt;mirror&gt;
      &lt;id&gt;maven.net.cn&lt;/id&gt;
      &lt;name&gt;one of the central mirrors in China&lt;/name&gt;
      &lt;url&gt;http://maven.net.cn/content/groups/public/&lt;/url&gt;
      &lt;mirrorOf&gt;central&lt;/mirrorOf&gt;
    &lt;/mirror&gt;
  &lt;/mirrors&gt;
  &lt;!--…--&gt;
&lt;/settings&gt;
[/code]

> **WARNING: **镜像仓库完全屏蔽了被镜像仓库，当镜像仓库不稳定或者不能访问的时候，Maven将无法访问被访问镜像。

	&nbsp;

> **参考资料**：《Maven实战》