title: Shell常用命令 文件/目录操作篇
tags:
  - Linux
  - Shell
id: 637
categories:
  - 技术分享
date: 2013-01-18 20:38:24
---

**2.1 ****基本操作**

	ls(list), cd(change directory), rm(remove), touch, mkdir(make directory), rmdir(remove directory), pwd(print working dir)

> **TIPS: **cd - 可以返回上次所处目录。> 
> 
> 		**WARNING:** 慎用rm -rf *，想当年某同学就rm掉了工作目录，不过代码有备份。

	<!--more-->

	&nbsp;

	**2.2 ****文件查找**

	最常用的当属**find**命令了。find的使用格式及我常用例子如下：

<table border="1" cellpadding="0" cellspacing="0" width="631">
	<tbody>
		<tr>
			<td valign="top" width="629">

					&nbsp; &gt;&gt; $ find -help

					&nbsp; Usage: find [path] [expression](&lt;option&gt; &lt;action&gt;) # commented by wikie

					&nbsp; &gt;&gt; $ find . -name &quot;*.txt&quot;&nbsp;&nbsp;&nbsp; # 查找当前目录下所有符合&quot;*.txt&quot;的文件

					&nbsp; &gt;&gt; $ find . -type f|d|l&nbsp;&nbsp;&nbsp; # 查找当前目录下所有普通文件|目录|链接符号

					&nbsp; &gt;&gt; $ find . -size 1024(c)&nbsp;&nbsp;&nbsp; # 查找当前目录大小为1024个块（字节）的文件

					&nbsp; &gt;&gt; $ find . -maxdepth 1 -name &quot;*.txt&quot;&nbsp;&nbsp;&nbsp; # 设置最大查找深入，1表示为当前目录

					&nbsp; &gt;&gt; $ find . -a(m)time -1&nbsp;&nbsp;&nbsp; # 查找当前目录下最后24小时访问（修改）的文件

					&nbsp; &gt;&gt; $ find . -name &quot;*.tmp&quot; -exec(-ok) rm {} \;&nbsp;&nbsp; # 删除当前目录的所有tmp文件，-ok表示执行前先询问用户

			</td>
		</tr>
	</tbody>
</table>

> **TIPS1:** 如不太喜欢用-exec，也可以用 find . | while read line; do do_sth to $line; done;> 
> 
> 		**TIPS2:** 最好的参考文档莫过于man find了

	**locate****：**

	locate命令其实是&ldquo;find -name&rdquo;的另一种写法，但是要比后者快得多，原因在于它不搜索具体目录，而是搜索一个数据库（/var/lib/locatedb），这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不到最新变动过的文件。为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库。不过需要root权限。

<table border="1" cellpadding="0" cellspacing="0" width="631">
	<tbody>
		<tr>
			<td valign="top" width="629">

					&nbsp; &gt;&gt; $ locate your_file

			</td>
		</tr>
	</tbody>
</table>

	**whereis:**

	whereis命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。

<table border="1" cellpadding="0" cellspacing="0" width="632">
	<tbody>
		<tr>
			<td valign="top" width="630">

					&nbsp; &gt;&gt; $ whereis whereis

					&nbsp; whereis: /usr/bin/whereis /usr/share/man/man1/whereis.1.gz

			</td>
		</tr>
	</tbody>
</table>

	**which:**

	which命令的作用是，在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。

<table border="1" cellpadding="0" cellspacing="0" width="633">
	<tbody>
		<tr>
			<td valign="top" width="631">

					&nbsp; &gt;&gt; $ which gcc

					&nbsp; /usr/bin/gcc

			</td>
		</tr>
	</tbody>
</table>

> **REFE: **以上多数文字摘抄自阮一峰的文章 - 《[Linux的五个查找命令](http://www.ruanyifeng.com/blog/2009/10/5_ways_to_search_for_files_using_the_terminal.html)》

	&nbsp;

	**2.3 ****磁盘操作**

	du(disk usage)查看目录大小，df(disk free)查看磁盘状况。均有-h参数，代表human-readable，即数据小大会显示单位。

<table border="1" cellpadding="0" cellspacing="0" width="634">
	<tbody>
		<tr>
			<td valign="top" width="632">

					&nbsp; &gt;&gt; $ du -sh .&nbsp;&nbsp;&nbsp;&nbsp; # -s summay

					&nbsp; 10G&nbsp;&nbsp;&nbsp;&nbsp; .

					&nbsp; &gt;&gt; $ df -h

					&nbsp; Filesystem&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Size&nbsp;&nbsp; Used&nbsp;&nbsp; Avail&nbsp;&nbsp; Use%&nbsp;&nbsp; Mounted on

					&nbsp; /dev/hda1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 62G&nbsp;&nbsp;&nbsp; 21G&nbsp;&nbsp;&nbsp; 42G&nbsp;&nbsp;&nbsp; 34%&nbsp;&nbsp;&nbsp;&nbsp; /

					&nbsp; tmpfs&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 502M&nbsp;&nbsp;&nbsp; 0&nbsp;&nbsp;&nbsp;&nbsp; 502M&nbsp;&nbsp;&nbsp; 0%&nbsp;&nbsp;&nbsp;&nbsp; /dev/shm

					&nbsp; &gt;&gt; $ df -ih&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; # -i inode usage

					&nbsp; Filesystem&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Inodes&nbsp;&nbsp; IUsed&nbsp;&nbsp; IFree&nbsp;&nbsp; IUse%&nbsp;&nbsp; Mounted on

					&nbsp; /dev/hda1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 63M&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 197K&nbsp;&nbsp;&nbsp; 62M&nbsp;&nbsp;&nbsp; 1%&nbsp;&nbsp; /

					&nbsp; tmpfs&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 126K&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 126K&nbsp;&nbsp;&nbsp; 1%&nbsp;&nbsp; /dev/shm

			</td>
		</tr>
	</tbody>
</table>

> **WARNING: **这两个命令非常重要，在机器上运行大任务的同时，切忌注意磁盘空间是否充裕（我们这里规定超过85%就报警）。经常有同事撑爆服务器，上头就过来找我麻烦T_T，真的冤枉。话说，以前也因为没有估量自己任务大小，撑爆一个集群的情况出现&hellip;&hellip;> 
> 
> 		**TIPS:** df是针对整个文件系统的，它通过系统调用statfs从文件系统的超级块中获取整个文件系统的磁盘使用情况，它没有没法针对任意目录来统计，但统计磁盘大小速度更快。如果需要统计磁盘大小，可以用df代替du：df -h | grep mounted_disk | awk &#39;{print $4}&#39;

	&nbsp;

	**2.4 ****权限管理**

	chmod(change mode), chown(change owner)

	说实在的，这两个命令在非管理员的情况下一般用不到。用得多的是chmod +x file，给文件添加执行权限。其他的还有需求自行谷歌。

	&nbsp;

	**2.5 ****文件压缩、裁剪**

	压缩用的是tar命令，也有用zip的但是不多。tar的参数实在是太多了，列不过来，就说两个常用的压缩解压缩吧：

<table border="1" cellpadding="0" cellspacing="0" width="630">
	<tbody>
		<tr>
			<td valign="top" width="628">

					&nbsp; &gt;&gt; $ tar zcvf file.tar.gz file&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; # z 以gzip的压缩方式压缩&nbsp; c 压缩

					&nbsp; &gt;&gt; $ tar zxvf file.tar.gz [-C untar_dir]

			</td>
		</tr>
	</tbody>
</table>

> **WARNING: **需要注意gzip压缩解压缩非常耗时且耗cpu> 
> 
> 		**TIPS: **在远程拷贝（scp）文件的时候，如果文件非常大，需要考虑拷贝中断所带来的痛苦。所以适当的将文件裁剪成多个再scp，拷贝中断带来的痛苦系数会大大减小。

<table border="1" cellpadding="0" cellspacing="0" width="632">
	<tbody>
		<tr>
			<td valign="top" width="630">

					&nbsp; &gt;&gt; $ split --help

					&nbsp; Usage: split [OPTION] [INPUT [PREFIX]]

					&nbsp; &gt;&gt; $ split -b 1024b(k)(m) filename (splited_filename)&nbsp;&nbsp; # 按大小裁剪

					&nbsp; &gt;&gt; $ split -l 1000 filename (splited_filename)&nbsp;&nbsp; # 按行数裁剪

					&nbsp; &gt;&gt; $ cat splited_filename* &gt; filename&nbsp;&nbsp; # 合并被裁剪文件

			</td>
		</tr>
	</tbody>
</table>