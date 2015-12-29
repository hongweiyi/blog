title: Shell常用命令 终端优化前篇
tags:
  - Linux
  - Shell
id: 630
categories:
  - 技术分享
date: 2013-01-17 23:18:50
---

在黑框框shell下作业也一年多了，其中实战占了9个月，从最开始的翻阅各种书籍，到后面问谷歌，也算是能熟练&ldquo;掌握&rdquo;这门工具，也搜集了一些&ldquo;奇技淫巧&rdquo;。现在好好的总结一下吧，顺带也查漏补缺&hellip;&hellip;

	<!--more-->

	**1.1 ****终端设置**

	下载一个好用的终端，如**[xshell](http://www.netsarang.com/products/xsh_overview.html)**，[**putty**](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)。

	设置Encoding、Terminal Type，一般来说在终端中设置一下即可。Encoding为utf8，Terminal Type设置为xterm、VT100、linux都可以，设置其他的类型有的可能ls文件会没有颜色（这个情况在screen命令中也会出现，到时候再讲解决方案吧），或者Home/End键不好用（代替快捷键：Ctrl+a/e）之类的。

	**1.2 shell****提示符设置**

	终端提示符默认的就是白色的用户名，看着非常不舒服，可以设置一下当前环境的提示符颜色，变量是PS1：

	[![image](http://hongweiyi.com/wp-content/uploads/2013/01/image_thumb.png "image")](http://hongweiyi.com/wp-content/uploads/2013/01/image.png)

<table border="1" cellpadding="0" cellspacing="0" width="626">
	<tbody>
		<tr>
			<td valign="top" width="624">

					&nbsp; &gt;&gt; $ echo $PS1

					&nbsp; \[\e[31;1m\]\u\[\e[0m\]@\[\e[32;1m\]your ip\[\e[0m\]:\[\e[35;1m\]\w\[\e[0m\]\\$

			</td>
		</tr>
	</tbody>
</table>

	你也可以不填写ip，用命令获得当前ip：

<table border="1" cellpadding="0" cellspacing="0" width="626">
	<tbody>
		<tr>
			<td valign="top" width="624">

					&nbsp; &gt;&gt; $ export PS1=&quot;\[\e[31;1m\]\u\[\e[0m\]@\[\e[32;1m\]`/sbin/ifconfig eth1|grep "inet&nbsp;&nbsp; addr:"|cut -d: -f 2|cut -d" " -f1`\[\e[0m\]:\[\e[35;1m\]\w\[\e[0m\]\\$ &quot;

			</td>
		</tr>
	</tbody>
</table>

	其他颜色可参考：**[Linux终端下的颜色设置](http://unix-cd.com/unixcd12/article_7281.html)**

	上面的PS1变量可以设置在用户配置中，文件为~/.bashrc，在该文件中添加一行：export PS1=&times;&times;&times;即可。如果再次登陆的时候，该命令没有生效，则添加~/.bash_profile文件，当shell被打开时，该文件会被执行一次。在文件中添加命令即可加载~/.bashrc：

<table border="1" cellpadding="0" cellspacing="0" width="625">
	<tbody>
		<tr>
			<td valign="top" width="623">

					&nbsp; # Get the aliases and functions

					&nbsp; if [ -f ~/.bashrc ]; then

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; . ~/.bashrc

					&nbsp; fi

			</td>
		</tr>
	</tbody>
</table>

	如果你想该文件立即生效，可以用source ~/.bashrc命令立即加载执行该文件。

	&nbsp;

	**1.3 ****别名设置**

	可以用一些简单的别名代替一些复杂的命令，如下：

<table border="1" cellpadding="0" cellspacing="0" width="626">
	<tbody>
		<tr>
			<td valign="top" width="624">

					&nbsp; alias ..=&quot;cd ..&quot;

					&nbsp; alias ...=&quot;cd ../..&quot;

					&nbsp;

					&nbsp; alias cd..=&quot;cd ..&quot;

					&nbsp; alias ll=&quot;ls -l&quot;

					&nbsp; alias la=&quot;ls -a&quot;

					&nbsp; alias tree=&quot;~/code/tree&quot;

			</td>
		</tr>
	</tbody>
</table>

	将alias添加进~/.bashrc中。~/code/tree是一个shell脚本，脚本内容如下：

<table border="1" cellpadding="0" cellspacing="0" width="626">
	<tbody>
		<tr>
			<td valign="top" width="624">

					&nbsp; #!/bin/sh

					&nbsp; function tree_files()

					&nbsp; {

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; find . -print 2&gt;/dev/null|\

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; awk &#39;!/\.$/ {for (i=1;i&lt;NF;i++){\

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; d=length($i);if ( d &lt; 5 &amp;&amp; i != 1 )d=5;printf(&quot;%&quot;d&quot;s&quot;,&quot;|&quot;)}print &quot;---&quot;$NF}&#39; \

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FS=&#39;/&#39; | more

					&nbsp; }

					&nbsp; function tree_dirs()

					&nbsp; {

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; find . -type d -print 2&gt;/dev/null|\

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; awk &#39;!/\.$/ {for (i=1;i&lt;NF;i++){\

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; d=length($i);if ( d &lt; 5 &amp;&amp; i != 1 )d=5;printf(&quot;%&quot;d&quot;s&quot;,&quot;|&quot;)}print &quot;---&quot;$NF}&#39; \

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FS=&#39;/&#39; | more

					&nbsp; }

					&nbsp; function help()

					&nbsp; {

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; echo &quot;Usage: tree [-f|-d]&quot;

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; echo &quot;&nbsp; -f: list files of current directory&quot;

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; echo &quot;&nbsp; -d: list direcotries of current directory&quot;

					&nbsp; }

					&nbsp; if [ $# = 0 ]; then

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; tree_dirs

					&nbsp; elif [ $# = 1 ]; then

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if [ $1 = "-f" ]; then

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; tree_files

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; elif [ $1 = "-d" ]; then

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; tree_dirs

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; else

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; help

					&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; fi

					&nbsp; fi

			</td>
		</tr>
	</tbody>
</table>

	&nbsp;

	**1.4 ****文档编辑器设置**

	我用的是vim，配置文件为~/.vimrc。由于工作机较多，所以没有一个个去配置，不可缺少的是syntax on（开启高亮），indent都没太大必要了。当然，专用开发机需要配置一个比较好用的，因人而异，如需要较好配置文件的可自行谷歌。

	有了以上四步，基本上一个简单方便好用的终端就有了，接下来我会一一介绍常用的shell命令：文件/目录操作、输入/输出操作、文本操作、进程操作等。

> **BTW：**个人不太喜欢装各种各样的辅助工具（tmux、vim相关、make相关。tree的工具也不想装，直接写shell多方便），因为环境多变，如果突然没了插件可能会不知所措，喜欢最原始最朴素的命令和工具，认为这才是王道！