title: Shell常用命令 文本操作篇
tags:
  - Linux
  - Shell
id: 647
categories:
  - 技术分享
date: 2013-01-19 22:41:44
---

### 3.0 前言

《Linux/Unix设计思想》中有一条准则：采用纯文本文件来存储数据，原因如下：

1. 文本是通用的可转换格式2. 文本文件易于阅读和编辑
3. 文本数据文件简化了Unix工具的使用
4. 可移植性的提高克服了速度的不足
5. 速度欠佳的缺点会被明年的机器克服

在实战中有能感受到，文本文件的方便，也有非常多的工具辅助开发者处理、查看、编辑它们，如：awk、sed等，这篇博文就介绍与文本操作相关的常用命令。

<!--more-->

### 3.1 输入输出流

#### echo：

echo如同python中的print，不过不需要引号包住字符串，echo后接的除管理和重定向字符外均会输出。

#### cat（Concatenate）：

它主要的功能是将文件的内容连续（concatenate）的输出到屏幕上，在上篇博文中就已经看到了，用来合并多个split的文件。

#### |：

管道符号，接收管道左边命令的输出，并输入到右边的命令。在Linux中，管道是一种使用非常频繁的通信机制。从本质上说，管道也是一种文件，但它又和一般的文件有所不同。在 Linux 中，管道的实现并没有使用专门的数据结构，而是借助了文件系统的file结构和VFS的索引节点inode。通过将两个 file 结构指向同一个临时的 VFS 索引节点，而这个 VFS 索引节点又指向一个物理页面而实现的。

这样，左边命令将数据写到索引节点，右边读取索引节点，读完之后方能继续写。详细实现机制可参考：[Linux管道的实现机制](http://oss.org.cn/kernel-book/ch07/7.1.1.htm)

#### 输出重定向：

`>`: 将左边的输出输入到右边的文件

`>>`: 将左边命令的输出追加输入到右边的文件

#### 标准输出流重定向：

0、1、2分别代表了标准输入、标准输出、标准错误信息输出，实战中经常会将错误输出重定向到标准输出中，可用 2>&1。同时，linux中有特殊的文件，/dev/null可理解为回收站，所有输出输入到该文件中均会“消失”。

#### read:

`Usage: read [option] [var(default:REPLY)]`

从标准输出流中读取一行，并给一个变量赋值。

```
>> $ echo "hello world" > new.txt
>> $ cat new.txt
hello world
>> $ echo "again" >> new.txt
>> $ cat new.txt
hello world
again
>> $ echo "col1 col2" | awk '{print $1}'
col1
>> $ read newline < new.txt; echo $newline
hello world
>> $ read < new.txt; echo $REPLY
hello world
```

> TIPS1: read最常见的应该是用在while循环中了，cat file | while read line; do do_sth; done;>
> TIPS2: 2>&1常见的地方是用在定时任务中，将所有输出结果均输出到/dev/null中，/bin/sh your.sh > /dev/null 2>&1
>
> TIPS3: Linux中最快的创建空文件的方法不是touch，而是"> newfile"

### 3.2 文本查看

cat、head、tail、more、less均是查看文本的好工具，有时候结合起来效果会更好。

#### head(tail)：

查看文件首（尾）十行

#### more：

先显示文件一屏，再等待用户输入，回车是再显示下一行，空格是下一屏，q(ctrl+c)则退出

#### less：

类似more，不过可以往上翻。可用上下键，也可以用u(up)，d(down)键。

```
>> $ tail -n 1000 file | more    # 查看文件的最后1k行
>> $ head -n 10010 file | tail     # 查看文件的第10k行
>> $ head -c 1(b|k|m) file    # 查看文件的前1B|K|M数据
```


### 3.3 文本字符集

字符集一直开发者的一个痛，按照东犇的说法是，要废除除utf8外所有字符集。此话虽然暴力，但是也有道理，不过要实现这个想法也基本是不太可能。开发者就只能一步一个脚印的走了。

#### file:

这是查看文本字符串的命令，不过个人觉得，基本不靠谱。因为要查看的数据都是代码生成的，输出的数据也不会有BOM信息，file也没有实现自检测文本字符功能。所以，现实中一般就是连蒙带猜了，不是gbk就是utf8。哈哈

#### iconv:

字符集转换工具

```
>> $ iconv -f from_code -t to_code file
```

文本字符集转换在这篇文章就只能点到为止了，字符集的问题三两句话基本讲不清楚，改天有机会单独写篇文章讲讲。

> TIPS1: 输出文本到屏幕，显示的字符集是由终端所设置的。当出现乱码的时候，不要误认为是数据出错了，很有可能就是你的字符集设置有问题。>
>
> TIPS2: 按照东犇的说法，建议开发者统一使用utf8编码。他也提供了一种较为方便的查看gbk文本的方法。```
>> $ echo -e "\n"alias conv_gbk=\"iconv -f gbk -t utf8\" >> ~/.bashrc>> $ source ~/.bashrc>> $ cat file | conv_gbk```

### 3.4 文本其他工具

#### grep:

查找命令，是开发中使用频率非常高的命令。像我们这边，代码就是文档，最好的翻阅文档工具就是grep了。

```
>> $ grep [pattern] [file]    # grep是基于正则表达式的，查找file中符合pattern的行
>> $ grep -v pattern file    # 查找file中不符合pattern的行
>> $ grep pattern . -r    # -r recursive 查找当前目录下所有符合pattern的文件
```

#### wc:

wordcount工具，表示中国文字基本用不到。可以用其中一个：

```
>> $ wc -l file    # 统计file的行数
>> $ ls . | wc -l    # 统计当前目录下文件的数目
>> $ grep pattern file | wc -l    # 统计file中符合pattern的行数
```

#### sort:

排序工具：

```
>> $ sort -k n (-n) (-r) file   # 按照第n列，给file排序。-n numeric-sort -r reverse
>> $ ls -l . | sort -k 5 -n    # 列出当前目录的文件，并按文件大小排序
>> $ ls -l . | sort -k5n -k6    # 列出当前目录的文件，并按文件大小及时间排序
```

#### uniq:

去除连续重复行：

```
>> $ cat file
test1
test2
test2
test1
>> $ uniq file
test1
test2
test1
>> $ sort file | uniq     # 去除所有重复行
test1
test2
```

#### diff & patch:

文本比较工具，和svn里面的diff一样。diff的文件较小的话，结果还可以忍受，如果过大，那结果就不是给人看的了，需要让机器来做一些工作了，如做增量拷贝。这个命令就是patch。

命令较为繁琐，且个人用得较少，就直接引用其他文章了：[用diff和patch工具维护源码](https://www.ibm.com/developerworks/cn/linux/l-diffp/)

#### awk & sed:

sed(stream editor)意为流编辑器，我的理解就是它是按行处理。而awk则不仅仅可以按行，还可以按列。想必前面的文章，也看到了不少用awk的命令。两者各有所长，且博大精深啊，也有关于这两个工具的书。笔者在这里也只介绍我常用到的功能。

```
>> $ echo "col1 col2 col3" | awk '{print $1"\t"$3}'    # $0：输入串 $1：col1 $2：col2 $3：col3 …
col1[tab]col3
>> $ echo "col1 col2 col3" | awk '{print $1, $3}'
col1[space]col3
>> $ sed -i "s/pattern/replace_pattern/g" file    # 替换字符串
```

> WARNING: 当文本为中文的时候，sed -i "s/char/replace_char/g"替换ascii标点的时候，需要特别注意。因为gbk编码中有些字节会小于128，而sed替换也是逐字节对比替换，所以有时候会导致中文乱码。
