title: OOM Killer 的一次问题定位
tags:
  - java
  - JVM
  - OOM Killer
id: 1125
categories:
  - 技术分享
date: 2015-03-16 22:01:01
---

这两天为了节省服务器资源，讲多个不同的 JVM 部署到了同一个 VM 上，想着应该没什么事，大不了处理速度慢一点而已，但是没想到确出现了意想不到的状况：各个 VM 上的 JVM 不约而同的挂了。挂了没事，解决 Bug 嘛，但是问题在于 JVM 是怎么挂掉了就没有搞清楚，也没有特殊日志打印，我花了半天时间定位了问题。<!--more-->

## JVM 为什么挂掉了？

正常来说，JVM 挂了要么会生成内存 dump ，要么直接生成 core 文件，我的机器什么都没有产生。于是乎只能借助系统工具了，如下命令能够捕获进程信号：

[code lang="shell"]
strace -e trace=signal -o /home/admin/strace.log -p [PID];
[/code]

等了几个小时后，有 JVM 挂了，日志输出如下：

[code lang="shell"]+++ killed by SIGKILL +++[/code]

[signal - overview of signals](http://man7.org/linux/man-pages/man7/signal.7.html) 这里可以看到，SIGKILL 就是 kill -9，进程不会做任何处理直接退出。看到这个我以为咱们的 OS 部署了监控进程，会用来 kill 掉耗资源的进程，咨询运维人员后，没有这样的程序部署，排除他方的因素，自身程序有问题可能性较大。

从信号上看不出什么端倪，就只能从系统日志上面来找了，通过以下命令发现 JVM 挂掉的原因：

[code lang="shell"]
dmesg | egrep -i -B100 'killed process'
[5673702.665338] [20189]   522 20189     1017       22   2       0             0 sleep
[5673702.665338] [20308]     0 20308    47967    20414   3       0             0 puppet
[5673702.665338] [20536]     0 20536    47969    20419   1       0             0 puppet
[5673702.665338] Out of memory: Kill process 29953 (java) score 431 or sacrifice child
[5673702.665338] Killed process 29953, UID 500, (java) total-vm:9805316kB, anon-rss:2344496kB, file-rss:128kB
[/code]

是由于 Out of memory 导致 JVM 被直接 kill 掉，这也是较为常见的 OOM Killer 了，关于 OOM Killer 网上有篇不错的解析文章，摘抄见后文。

> 定位 OOM 具体问题，除了 dump 内存分析之外，还有一些较为简单快捷的方式对整个内存进行一次摸底。
> 
> pmap -x [PID]: 能查看进程的内存映射;
> 
> jmap -heap [PID]: 快速查看 JVM 各内存区域的使用情况。

## 理解和配置 Linux 下的 OOM Killer

最近有位 VPS 客户抱怨 MySQL 无缘无故挂掉，还有位客户抱怨 VPS 经常死机，登陆到终端看了一下，都是常见的 Out of memory 问题。这通常是因为某时刻应用程序大量请求内存导致系统内存不足造成的，这通常会触发 Linux 内核里的 Out of Memory (OOM) killer，OOM killer 会杀掉某个进程以腾出内存留给系统用，不致于让系统立刻崩溃。如果检查相关的日志文件（/var/log/messages）就会看到下面类似的 Out of memory: Kill process 信息：

[code lang="shell"]
...
Out of memory: Kill process 9682 (mysqld) score 9 or sacrifice child
Killed process 9682, UID 27, (mysqld) total-vm:47388kB, anon-rss:3744kB, file-rss:80kB
httpd invoked oom-killer: gfp_mask=0x201da, order=0, oom_adj=0, oom_score_adj=0
httpd cpuset=/ mems_allowed=0
Pid: 8911, comm: httpd Not tainted 2.6.32-279.1.1.el6.i686 #1
...
21556 total pagecache pages
21049 pages in swap cache
Swap cache stats: add 12819103, delete 12798054, find 3188096/4634617
Free swap  = 0kB
Total swap = 524280kB
131071 pages RAM
0 pages HighMem
3673 pages reserved
67960 pages shared
124940 pages non-shared
[/code]

Linux 内核根据应用程序的要求分配内存，通常来说应用程序分配了内存但是并没有实际全部使用，为了提高性能，这部分没用的内存可以留作它用，这部分内存是属于每个进程的，内核直接回收利用的话比较麻烦，所以内核采用一种过度分配内存（over-commit memory）的办法来间接利用这部分 “空闲” 的内存，提高整体内存的使用效率。一般来说这样做没有问题，但当大多数应用程序都消耗完自己的内存的时候麻烦就来了，因为这些应用程序的内存需求加起来超出了物理内存（包括 swap）的容量，内核（OOM killer）必须杀掉一些进程才能腾出空间保障系统正常运行。用银行的例子来讲可能更容易懂一些，部分人取钱的时候银行不怕，银行有足够的存款应付，当全国人民（或者绝大多数）都取钱而且每个人都想把自己钱取完的时候银行的麻烦就来了，银行实际上是没有这么多钱给大家取的。

内核检测到系统内存不足、挑选并杀掉某个进程的过程可以参考内核源代码 [linux/mm/oom_kill.c](https://github.com/torvalds/linux/blob/master/mm/oom_kill.c)，当系统内存不足的时候，out_of_memory() 被触发，然后调用 select_bad_process() 选择一个 “bad” 进程杀掉，如何判断和选择一个 “bad” 进程呢，总不能随机选吧？挑选的过程由 oom_badness() 决定，挑选的算法和想法都很简单很朴实：最 bad 的那个进程就是那个最占用内存的进程。

[code lang="c"]
/**
 * oom_badness - heuristic function to determine which candidate task to kill
 * @p: task struct of which task we should calculate
 * @totalpages: total present RAM allowed for page allocation
 *
 * The heuristic for determining which task to kill is made to be as simple and
 * predictable as possible.  The goal is to return the highest value for the
 * task consuming the most memory to avoid subsequent oom failures.
 */
unsigned long oom_badness(struct task_struct *p, struct mem_cgroup *memcg,
			  const nodemask_t *nodemask, unsigned long totalpages)
{
	long points;
	long adj;

	if (oom_unkillable_task(p, memcg, nodemask))
		return 0;

	p = find_lock_task_mm(p);
	if (!p)
		return 0;

	adj = (long)p-&amp;gt;signal-&amp;gt;oom_score_adj;
	if (adj == OOM_SCORE_ADJ_MIN) {
		task_unlock(p);
		return 0;
	}

	/*
	 * The baseline for the badness score is the proportion of RAM that each
	 * task's rss, pagetable and swap space use.
	 */
	points = get_mm_rss(p-&amp;gt;mm) + p-&amp;gt;mm-&amp;gt;nr_ptes +
		 get_mm_counter(p-&amp;gt;mm, MM_SWAPENTS);
	task_unlock(p);

	/*
	 * Root processes get 3% bonus, just like the __vm_enough_memory()
	 * implementation used by LSMs.
	 */
	if (has_capability_noaudit(p, CAP_SYS_ADMIN))
		adj -= 30;

	/* Normalize to oom_score_adj units */
	adj *= totalpages / 1000;
	points += adj;

	/*
	 * Never return 0 for an eligible task regardless of the root bonus and
	 * oom_score_adj (oom_score_adj can't be OOM_SCORE_ADJ_MIN here).
	 */
	return points &amp;gt; 0 ? points : 1;
}
[/code]

上面代码里的注释写的很明白，理解了这个算法我们就理解了为啥 MySQL 躺着也能中枪了，因为它的体积总是最大（一般来说它在系统上占用内存最多），所以如果 Out of Memeory (OOM) 的话总是不幸第一个被 kill 掉。解决这个问题最简单的办法就是增加内存，或者[想办法优化 MySQL 使其占用更少的内存](http://www.vpsee.com/2009/06/64mb-vps-optimize-mysql/)，除了优化 MySQL 外还可以优化系统（[优化 Debian 5](http://www.vpsee.com/2009/06/64mb-vps-optimize-debian5/)，[优化 CentOS 5.x](http://www.vpsee.com/2009/06/128mb-vps-optimize-centos5/)），让系统尽可能使用少的内存以便应用程序（如 MySQL) 能使用更多的内存，还有一个临时的办法就是调整内核参数，让 MySQL 进程不容易被 OOM killer 发现。

### 配置 OOM killer

我们可以通过一些内核参数来调整 OOM killer 的行为，避免系统在那里不停的杀进程。比如我们可以在触发 OOM 后立刻触发 kernel panic，kernel panic 10秒后自动重启系统。

[code lang="shell"]
# sysctl -w vm.panic_on_oom=1
vm.panic_on_oom = 1

# sysctl -w kernel.panic=10
kernel.panic = 10

# echo &quot;vm.panic_on_oom=1&quot; &amp;gt;&amp;gt; /etc/sysctl.conf
# echo &quot;kernel.panic=10&quot; &amp;gt;&amp;gt; /etc/sysctl.conf
[/code]

从上面的 oom_kill.c 代码里可以看到 oom_badness() 给每个进程打分，根据 points 的高低来决定杀哪个进程，这个 points 可以根据 adj 调节，root 权限的进程通常被认为很重要，不应该被轻易杀掉，所以打分的时候可以得到 3% 的优惠（adj -= 30; 分数越低越不容易被杀掉）。我们可以在用户空间通过操作每个进程的 oom_adj 内核参数来决定哪些进程不这么容易被 OOM killer 选中杀掉。比如，如果不想 MySQL 进程被轻易杀掉的话可以找到 MySQL 运行的进程号后，调整 oom_score_adj 为 -15（注意 points 越小越不容易被杀）：

[code lang="shell"]
# ps aux | grep mysqld
mysql    2196  1.6  2.1 623800 44876 ?        Ssl  09:42   0:00 /usr/sbin/mysqld

# cat /proc/2196/oom_score_adj
0
# echo -15 &amp;gt; /proc/2196/oom_score_adj
[/code]

当然，如果需要的话可以完全关闭 OOM killer（不推荐用在生产环境）：

[code lang="shell"]
# sysctl -w vm.overcommit_memory=2

# echo &quot;vm.overcommit_memory=2&quot; &amp;gt;&amp;gt; /etc/sysctl.conf
[/code]

### 找出最有可能被 OOM Killer 杀掉的进程

我们知道了在用户空间可以通过操作每个进程的 oom_adj 内核参数来调整进程的分数，这个分数也可以通过 oom_score 这个内核参数看到，比如查看进程号为981的 omm_score，这个分数被上面提到的 omm_score_adj 参数调整后（－15），就变成了3：

[code lang="shell"]
# cat /proc/981/oom_score
18

# echo -15 &amp;gt; /proc/981/oom_score_adj
# cat /proc/981/oom_score
3
[/code]

下面这个 bash 脚本可用来打印当前系统上 oom_score 分数最高（最容易被 OOM Killer 杀掉）的进程：

[code lang="shell"]
# vi oomscore.sh
#!/bin/bash
for proc in $(find /proc -maxdepth 1 -regex '/proc/[0-9]+'); do
    printf &quot;%2d %5d %s\n&quot; \
        &quot;$(cat $proc/oom_score)&quot; \
        &quot;$(basename $proc)&quot; \
        &quot;$(cat $proc/cmdline | tr '&#92;&#48;' ' ' | head -c 50)&quot;
done 2&amp;gt;/dev/null | sort -nr | head -n 10

# chmod +x oomscore.sh
# ./oomscore.sh
18   981 /usr/sbin/mysqld
 4 31359 -bash
 4 31056 -bash
 1 31358 sshd: root@pts/6
 1 31244 sshd: vpsee [priv]
 1 31159 -bash
 1 31158 sudo -i
 1 31055 sshd: root@pts/3
 1 30912 sshd: vpsee [priv]
 1 29547 /usr/sbin/sshd -D
[/code]

> 原文链接：http://www.vpsee.com/2013/10/how-to-configure-the-linux-oom-killer/