title: Shell常用命令 进程操作篇
tags:
  - Linux
  - Shell
id: 656
categories:
  - 技术分享
date: 2013-01-22 21:57:57
---

### 4.0 前言

这篇算是尾篇了，命令较多也较繁杂，每个都是点到为止，基本也够用。有兴趣深入的朋友可自行谷歌学习。

<!--more-->

### 4.1 进程执行

进程执行无外乎前台运行，但是前台运行需要用户等待程序运行完毕才可继续，所以就有了后台运行进程这一说。后台进程又分两种，当前终端后台进程，以及托管给OS的后台进程。这里就稍微介绍一下如何运行这些个进程吧。

> BTW: 以上分类都是我瞎分的，仅供参考。

#### 后台运行进程:

在运行命令末尾加上`&`符号，该程序就会在后台运行，如果忘记敲`&`命令的话，也可以同过ctrl+z，暂停当前进程，再用bg(backgroud)命令即可让暂停任务变成后台运行。

```
>> $ /bin/sh your.sh &
[1]    pid        # 输出进程在当前终端的序号以及pid
>> $ /bin/sh your.sh       # ctrl + z
[1]+  Stopped            /bin/sh your.sh
>> $ bg %1   # %1 代表当前终端的第一个进程
[1]+  /bin/sh your.sh &
```

> TIPS: The plus sign shows the most recently invoked job; the minus sign shows the next most recently invoked job.- <Learning the Korn Shell, 2nd Edition>>
> WARNING: 当运行某个有大量输出的进程，如果直接让其进入后台运行是很不明智的，一般来说会将输出写入文件中。如： ` $ /bin/sh your.sh > ./your.log 2>&1 &`

#### 前台运行进程：

有了后台，就有相应的前台运行进程了。fg(foreground)就是了，但一般来说，很少需要将后台进程推到前台的需求，我一般是这样用的：

```
>> $ vim your_file      # ctrl + z
[1]+   Stopped            vim your_file
>> $ fg %1
```

#### 托管进程：

上面的命令均是在当前终端运行，如果终端关闭，相应的进程也关闭了，如果想进程继续运行，则需要将进程托管给系统管理。主要命令是nohup。

```
>> $ nohup /bin/sh your.sh &
[1]    pid
nohup: appending output to ‘nohup.out'
```

实战的时候，更多的情况是这样的：突然有事要走开一下，又生怕跑了半天的程序因为终端挂了，想将其托管给系统。那么就可以用disown命令。

```
>> $ /bin/sh your.sh &
[1]    pid
>> disown %1
```

> 看这个解释得更为详细，我对其理解较为肤浅：[Linux技巧：让后台在后台可靠运行的几种方法](http://www.ibm.com/developerworks/cn/linux/l-cn-nohup/)。

#### screen:

上面两个命令都不是最好用的，最好用的应该是screen命令了。screen 提供了 ANSI/VT100 的终端模拟器，使它能够在一个真实终端下运行多个全屏的伪终端。screen用好了，感觉也不比tmux差多少。

要同时跑多个screen，并想快速切换的换，可以添加几个screen的alias。

```
alias s="screen"
alias sr="screen -r"     # 连接到某个模拟器
alias sl="screen -ls"     # 显示当前所有模拟器
alias sl="screen -d"     # 强制断开某个模拟器连接
ctrl+a :sessionname MyName   # 在screen模拟器中，修改模拟器名字
ctrl+a :kill    # 强制关闭模拟器（模拟器有时候会莫名其妙的没响应）
ctrl+a :encoding gbk     # 设置screen编码
ctrl+a d     # detached模拟器（就是临时退出）
```

> TIPS: screen终端ls文件没有颜色，是因为screen终端类型比较特殊，echo $TERM显示为screen或者screen.linux类型，解决方案是修改/etc/DIR_COLORS或者复制/etc/DIR_COLORS到~/.dir_colors，加入一句：&quot;TERM screen&quot;注销重登即可。

#### jobs:

列出当前终端运行的后台进程。

#### ps:

列出系统正在运行的程序。

#### top:

理解为win下面的任务管理器，需要注意的地方应该是load average了，Load Average表示了CPU的Load，它所包含的信息不是 CPU的使用率状况，而是在一段时间内CPU正在处理以及等待CPU处理的进程数之和的统计信息，也就是 CPU使用队列的长度的统计信息。load average: 0.06, 0.60, 0.48，三个数值分别为 1分钟、5分钟、15分钟前到现在的平均值。需要注意，load average如果超过一定的数的话，系统负载较高。

`Load Average < CPU个数 * 核数 *0.7`

我常用的命令也不多，如下：

```
>> $ top
…
…
>> $ top -p pid [-p pid2 …]
…    # 只查看pid这个进程
```

#### crontab:

定时任务命令，这个东西有时候也折磨了好一段时间。内容格式说着也简单：

```
>> $ crontab -e    # 编辑定时任务列表
cmd > /dev/null 2>&1     # 5个分别代表分、时、日、月、周。需要将输出写到某个文件或者/dev/null中，因为它的所有输出均会按邮件发到服务器上，日积月累也是个负担
/2 * cmd > /dev/null 2>&1     # 代表每两分钟执行一次cmd
>> $ crontab -l    # 显示定时任务列表
>> $ crontab -r    # 删除定时任务列表（- -，不知道为什么要有这个选项）
```

> 偷懒，直接贴其他人的帖子吧：[Crontab 错误分析及不执行原因](http://www.cnblogs.com/cosiray/archive/2012/03/09/2387361.html)

#### history:

查看命令历史，以下是常用方法和有用的设置：

```
>> $ history | more
>> $ history | grep ls
>> $ vi ~/.bashrc
HISTFILESIZE=2000     # history 记录长度
HISTTIMEFORMAT='%F %T '     # 给记录添加时间戳
```

> TIPS1: 输入命令前，输入一个空格，该记录不添加进history中>
> TIPS2: ctrl + r是找历史记录，查找过程中继续按ctrl+r是当前查找结果的上一条

#### ssh:

最后一个说ssh，好像也没啥说的。远程登录，远程拷贝

```
>> $ ssh -l user ip [-p port]
>> $ ssh user@ip
>> $ scp user@ip:/src/dir/or/file user@ip:/dest/dir/or/file
```

> FINAL TIPS: 拷贝的时候需要输入对方的用户密码，可以添加相关的密钥一劳永逸。但是添加密钥的方式有点微麻烦，所以可以用其他方式代替，那就是用expect命令。下面这段脚本可以实现无间断的远程push，可以自己改成远程pull。
``` sh
#!/usr/bin/expect
set user [lindex $argv 0]
set passwd [lindex $argv 1]
set ip [lindex $argv 2]
set src [lindex $argv 3]
set des [lindex $argv 4]
set timeout 60
spawn /usr/local/bin/scp -r ${src} ${user}@${ip}:${des}
expect {
    "*assword:" {
     send "${passwd}\n"
     exp_continue
    }
    "fcr_parse_raw" {
        close
        exit -2
    }
    timeout {
        close
        exit -1
    }
    eof {
        catch wait result
        exit [lindex $result 3]
    }
    -re . {
        exp_continue
    }
}
```
