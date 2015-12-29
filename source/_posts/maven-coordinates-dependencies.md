title: Maven笔记 – 坐标与依赖
tags:
  - java
  - Maven
id: 707
categories:
  - 技术分享
date: 2013-03-17 20:47:05
---

**1、前言**

	Maven ['meivin]，原意为内行、专家，用在Java开发中则是用于项目构建、依赖管理和项目信息管理。仔细总结一下，我们会发现，除了编写源代码，我们每天都有相当一部分时间花在了编译、运行单元测试、生成文档、打包和部署等繁琐且不起眼的工作上，这就是构建了。通过Maven可以通过一个简单的命令，让所有繁琐的步骤都能够自动完成，很方便的得到最终结果。

	<!--more-->

	这篇就介绍Maven比较重要的两个基本概念，坐标与依赖。以下大多数都是总结自《Maven实战》的笔记：

	&nbsp;

	**2、Maven****坐标元素**

	在开发项目的时候，开发者会到处收集第三方构件，而第三方构件版本各异并且难以收集，会导致大量时间花费在搜索、浏览网页等工作上。Maven则提供了一种统一的规范，让开发者通过简单的标识来表示一个构件并可以自动找到这个构件，这个标识就是坐标。通过以下坐标元素，可以唯一定义一个构件：

	**groupId****：**定义当前Maven项目隶属的实际项目

	**artifactId****：**定义实际项目中的一个Maven项目（模块），推荐使用实际项目名称作为artifactId的前缀，方便寻找实际构件。如nexus-indexer是nexus的indexer模块。

	**version****：**Maven项目当前所处的版本。

	**packaging(optional)****：**该元素定义Maven项目的打包方式（jar、war）

	**classsifier(optional)****：**附属构件，如javadocs、sources等，TestNG就有一个为jdk5的附属构件。BTW，不能直接定义项目的classifer，需要附加的插件帮助生成。

	[code lang="xml"]&lt;groupId&gt;com.hongweiyi&lt;/groupId&gt;
&lt;artifactId&gt;HelloWorld&lt;/artifactId&gt;
&lt;version&gt;0.0.1-SNAPSHOT&lt;/version&gt;
&lt;packaging&gt;jar&lt;/packaging&gt;

	[/code]

	&nbsp;

	**3、依赖配置**

	**groupId****、artifactId、version：**这三个基本坐标，必须有。

	**type****：**依赖的类型，对应于项目坐标定义的packaging，默认值为jar。

	**scope****：**依赖范围

	Maven在编译项目住代码、编译执行测试、实际运行项目的时候都会用相应的一套classpath

<table border="1" cellpadding="0" cellspacing="0">
	<tbody>
		<tr>
			<td valign="top">

					**依赖范围**

			</td>
			<td valign="top">

					**对于编译classpath有效**

			</td>
			<td valign="top">

					**对于测试classpath有效**

			</td>
			<td valign="top">

					**对于运行时classpath有效**

			</td>
			<td valign="top">

					**例子**

			</td>
		</tr>
		<tr>
			<td valign="top">

					compile

			</td>
			<td valign="top">

					**Y**

			</td>
			<td valign="top">

					**Y**

			</td>
			<td valign="top">

					**Y**

			</td>
			<td valign="top">

					spring-core

			</td>
		</tr>
		<tr>
			<td valign="top">

					test

			</td>
			<td valign="top">

					-

			</td>
			<td valign="top">

					**Y**

			</td>
			<td valign="top">

					**-**

			</td>
			<td valign="top">

					JUnit

			</td>
		</tr>
		<tr>
			<td valign="top">

					provided

			</td>
			<td valign="top">

					**Y**

			</td>
			<td valign="top">

					**Y**

			</td>
			<td valign="top">

					**-**

			</td>
			<td valign="top">

					servlet-api

			</td>
		</tr>
		<tr>
			<td valign="top">

					runtime

			</td>
			<td valign="top">

					**-**

			</td>
			<td valign="top">

					**Y**

			</td>
			<td valign="top">

					**Y**

			</td>
			<td valign="top">

					JDBC 驱动实现

			</td>
		</tr>
		<tr>
			<td valign="top">

					system

			</td>
			<td valign="top">

					**Y**

			</td>
			<td valign="top">

					**Y**

			</td>
			<td valign="top">

					**-**

			</td>
			<td valign="top">

					本地的，Maven仓库之外的类库文件

			</td>
		</tr>
		<tr>
			<td valign="top">

					import

			</td>
			<td valign="top">

					**-**

			</td>
			<td valign="top">

					**-**

			</td>
			<td valign="top">

					**-**

			</td>
			<td valign="top">

					导入依赖范围，并无实际影响

			</td>
		</tr>
	</tbody>
</table>

	&nbsp;

	传递依赖，在使用某开源项目的时候，经常使用了某三方包后，发现该三方包还依赖其他包，这就不得不一个个导入相应的依赖包。而Maven则很方便，不用考虑三方包依赖了什么，Maven会直接解析各个直接依赖的POM，并将那些必要的间接依赖，以传递性依赖的形式引入到当前的项目中。

	传递依赖也会有问题，比如：A-&gt;B-&gt;C-&gt;X(1.0)；A-&gt;D-&gt;X(2.0)，在这种情况下，Maven依赖解调第一原则是：路径最近者优先。上面那个例子，X(2.0)路径较近，所以会优先解析它。那么如果路径长度一样怎么解析呢？如：A-&gt;B-&gt;X(1.0) A-&gt;C-&gt;X(2.0)，这个就要用Maven解调的第二原则：第一声明者优先。在路径长度相等的前提下，POM中依赖声明的顺序决定了谁会被解析使用。

> BTW：依赖范围不仅可以控制依赖与三种classpath的关系，还对依赖性传递产生影响，这个影响有点绕，笔者还没弄太明白，就先不发了。T_T

	**optional****：**可选依赖。 (false / true)

	A-&gt;B, B-&gt;X(optional), B-&gt;Y(optional)

	A如果要用X或者Y的话，需要在POM中显示声明该Dependency。

	[code lang="xml"]&lt;project&gt;
&lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;
&lt;dependecies&gt;
   &lt;dependecy&gt;
      &lt;groupId&gt;com.hongweiyi.blog&lt;/groupId&gt;
      &lt;artifactId&gt;project-b&lt;/artifactId&gt;
      &lt;version&gt;1.0.0&lt;/version&gt;
   &lt;/dependency&gt;
   &lt;dependency&gt;        &lt;!-- 显示声明X (Y) --&gt;
     &lt;groupId&gt;com.hongweiyi.blog&lt;/groupId&gt;
     &lt;artifactId&gt;project-X&lt;/artifactId&gt; &lt;!--artifactId&gt;project-Y&lt;/artifactId--&gt;
     &lt;version&gt;1.1.0&lt;/version&gt;
   &lt;/dependency&gt;
&lt;/dependencies&gt;
&lt;/project&gt;

	[/code]

	**exclusions：**排除传递性依赖。

	传递性会给项目隐式地引入很多依赖，简化项目依赖管理的同时，也会带来一些风险。如果有一个三方包依赖，而这三方包依赖了另一个类库的SNAPSHOT版本，这个版本就成为了当前项目的传递性依赖，而SNAPSHOT的不稳定性也会直接影响到当前项目。所以需要将这个SNAPSHOT的版本给排除掉，并显示声明一个好用的版本。

	[code lang="xml"]&lt;project&gt;
&lt;modelVersion&gt;4.0.0&lt;/modelVersion&gt;
&lt;dependecies&gt;
   &lt;dependecy&gt;
      &lt;groupId&gt;com.hongweiyi.blog&lt;/groupId&gt;
      &lt;artifactId&gt;project-b&lt;/artifactId&gt;
      &lt;version&gt;1.0.0&lt;/version&gt;
      &lt;exclusions&gt;
         &lt;exclusion&gt;      &lt;!-- 排除project-c --&gt;
            &lt;groupId&gt;com.hongweiyi.blog&lt;/groupId&gt;
            &lt;artifactId&gt;project-c&lt;/artifactId&gt;
          &lt;/exclusion&gt;
      &lt;exclusions&gt;
   &lt;/dependency&gt;
   &lt;dependency&gt;        &lt;!-- 重新声明project-c 1.1.0版 --&gt;
     &lt;groupId&gt;com.hongweiyi.blog&lt;/groupId&gt;
     &lt;artifactId&gt;project-c&lt;/artifactId&gt;
     &lt;version&gt;1.1.0&lt;/version&gt;
   &lt;/dependency&gt;
&lt;/dependencies&gt;
&lt;/project&gt;

	[/code]

	&nbsp;

> 参考资料：《Maven实战》