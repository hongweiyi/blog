title: Shell常用命令 文件/目录操作篇
tags:
  - Linux
  - Shell
id: 637
categories:
  - 技术分享
date: 2013-01-18 20:38:24
---

### 2.1 基本操作

ls(list), cd(change directory), rm(remove), touch, mkdir(make directory), rmdir(remove directory), pwd(print working dir)

> TIPS: cd - 可以返回上次所处目录。> WARNING: 慎用rm -rf \*，想当年某同学就rm掉了工作目录，不过代码有备份。

<!--more-->


### 2.2 文件查找

#### find

最常用的当属find命令了。find的使用格式及我常用例子如下：

```
>> $ find -help
Usage: find [path] expression # commented by wikie
>> $ find . -name ".txt"    # 查找当前目录下所有符合".txt"的文件
>> $ find . -type f|d|l    # 查找当前目录下所有普通文件|目录|链接符号
>> $ find . -size 1024(c)    # 查找当前目录大小为1024个块（字节）的文件
>> $ find . -maxdepth 1 -name ".txt"    # 设置最大查找深入，1表示为当前目录
>> $ find . -a(m)time -1    # 查找当前目录下最后24小时访问（修改）的文件
>> $ find . -name ".tmp" -exec(-ok) rm {} \;   # 删除当前目录的所有tmp文件，-ok表示执行前先询问用户
```

> TIPS1: 如不太喜欢用-exec，也可以用 find . | while read line; do do_sth to $line; done;> TIPS2: 最好的参考文档莫过于man find了

#### locate

locate命令其实是`find -name`的另一种写法，但是要比后者快得多，原因在于它不搜索具体目录，而是搜索一个数据库（/var/lib/locatedb），这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不到最新变动过的文件。为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库。不过需要root权限。

```
>> $ locate your_file
```

#### whereis

whereis命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。

```
>> $ whereis whereis
whereis: /usr/bin/whereis /usr/share/man/man1/whereis.1.gz
```

#### which

which命令的作用是，在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。

```
>> $ which gcc
/usr/bin/gcc
```

> REFE: 以上多数文字摘抄自阮一峰的文章 - 《[Linux的五个查找命令](http://www.ruanyifeng.com/blog/2009/10/5_ways_to_search_for_files_using_the_terminal.html)》

### 2.3 磁盘操作

du(disk usage)查看目录大小，df(disk free)查看磁盘状况。均有-h参数，代表human-readable，即数据小大会显示单位。

```
>> $ du -sh .     # -s summay
10G     .
>> $ df -h
Filesystem            Size   Used   Avail   Use%   Mounted on
/dev/hda1             62G    21G    42G    34%     /
tmpfs                   502M    0     502M    0%     /dev/shm
>> $ df -ih          # -i inode usage
Filesystem            Inodes   IUsed   IFree   IUse%   Mounted on
/dev/hda1              63M      197K    62M    1%   /
tmpfs                    126K       1       126K    1%   /dev/shm
```

>  df是针对整个文件系统的，它通过系统调用statfs从文件系统的超级块中获取整个文件系统的磁盘使用情况，它没有没法针对任意目录来统计，但统计磁盘大小速度更快。如果需要统计磁盘大小，可以用df代替du：`df -h | grep mounted_disk | awk '{print $4}'`### 2.4 权限管理

chmod(change mode), chown(change owner)

说实在的，这两个命令在非管理员的情况下一般用不到。用得多的是chmod +x file，给文件添加执行权限。其他的还有需求自行谷歌。

### 2.5 文件压缩、裁剪

压缩用的是tar命令，也有用zip的但是不多。tar的参数实在是太多了，列不过来，就说两个常用的压缩解压缩吧：

```
>> $ tar zcvf file.tar.gz file                   # z 以gzip的压缩方式压缩  c 压缩
>> $ tar zxvf file.tar.gz [-C untar_dir]
```

> WARNING: 需要注意gzip压缩解压缩非常耗时且耗cpu> TIPS: 在远程拷贝（scp）文件的时候，如果文件非常大，需要考虑拷贝中断所带来的痛苦。所以适当的将文件裁剪成多个再scp，拷贝中断带来的痛苦系数会大大减小。

```
>> $ split –help
Usage: split [OPTION] [INPUT [PREFIX]]
>> $ split -b 1024b(k)(m) filename (splited_filename)   # 按大小裁剪
>> $ split -l 1000 filename (splited_filename)   # 按行数裁剪
>> $ cat splited_filename* > filename   # 合并被裁剪文件
```
