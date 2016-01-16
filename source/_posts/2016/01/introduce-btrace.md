title: BTrace 基本使用
date: 2016-01-13 22:54:00
categories: 最佳实践
tags: [Java, BTrace]
---

最近在做性能优化项目时，发现 RPC 框架调用极个别请求在路由过程非常耗时。对于这种极个别的问题要定位起来还是比较棘手的，如果性能问题可以稳定复现的还可以一步步打点缩小范围，但我这个是千分之一的出现概率，需要大规模日志打点才能够准确找到耗时的地方。如何搞呢，只能网上找找有没有更好的办法了，在网上看资料时，突然看到毕玄的文章提到可以用 btrace 定位这样的问题，现学现卖了一把。

<!--more-->

### 1. 基本使用

直接下载安装 [github release](https://github.com/jbachorik/btrace/releases/tag/v1.3.4) 即可。

* 添加 `BTRACE_HOME` 到环境变量
* 运行：`bin/btrace <PID> <trace_script>`
* 编译：`bin/btracec <trace_script>`

> btrace 不需要预先编译脚本。


### 2. 我的脚本


``` java
// 可以不用 package
package com.sun.btrace.scripts;

/* BTrace Script Template */
import com.sun.btrace.annotations.*;
import static com.sun.btrace.BTraceUtils.*;

@BTrace
public class ProfileScript {

    @TLS
    private static long startTime = 0;

    // com.alipay.sofa.rpc.process
    @OnMethod(clazz = "/com\\.hongweiyi\\.package\\.name\\..+/", method = "/.+/")
    public static void startMethod(){
        startTime = timeMillis();
    }

    @SuppressWarnings("deprecation")
    @OnMethod(clazz = "/com\\.hongweiyi\\.package\\.name\\..+/", method = "/.+/", location = @Location(Kind.RETURN))
    public static void endMethod() {

        long cost = timeMillis() - startTime;
        if (cost > 3) {
            print(strcat(strcat(strcat(strcat(name(probeClass()), "."), probeMethod()), ":"), str(probeLine())));
            print("  [");
            print(strcat("Time taken : ", str(timeMillis() - startTime)));
            println("]");
        }

    }
}
```

### 3. 使用实例

正常使用：启动程序，拿到 PID，运行 `bin/btrace PID ProfileScript.java`，在脚本目录会出现 `ProfileScript.xxx.btrace` 的输出日志，可以直接通过这个日志定位问题。

但是我的程序运行后不断抛 `java.lang.NoClassDefFoundError: ProfileScript` 异常。想到我的程序比较特殊，是运行在 OSGi 之上的，在网上搜到 btrace 在 glassfish 上运行同样有这样的问题。

如果需要将 btrace 运行在 OSGi 程序上，其中一个解决方案是配置 OSGi 的 bootdelegation：

`org.osgi.framework.bootdelegation = com.sun.btrace,com.sun.btrace.*`

> 注意：这个时候建议将 btrace 的 package 和 btrace 的设置成一样，这样 bootdelegation 才能生效

同时，通过 javaagent 配置 btrace 脚本：

`-javaagent:/home/admin/btrace/build/btrace-agent.jar=script=/home/admin/btrace/com/sun/btrace/scripts/ProfileScript.class`

> 注意：javaagent 的方式配置需要编译 btrace 脚本，不能直接使用 java 文件。编译命令 `bin/btracec com/sun/btrace/ProfileScript.java`

### 4. btrace 优化小结

如果 `@OnMethod(clazz)` 设置得比较准确的话，还是可以很快的定位一些优化点。优化了一些并发 Map 操作，性能提升 5%，发现了一个历史大坑，性能提升 90%。这个历史大坑也就是导致极个别请求耗时较长的原因。

我在压测过程中，YGC 比较频繁，这样会导致会出现很多不必要的结果。如果实在没有头绪了，建议把 `@OnMethod(clazz)` 设置的范围比较大，扫所有的的类，待拿到结果之后，排除掉一些噪音后可以看到一些不错的结果。运行：

`cat ProfileScript.btrace | awk '{print $1}' | sort | uniq -c | grep -v '1 ' | grep -v '2 ' | sort -k 2`

> 不要忽略任何一个可疑的问题，极个别耗时长的请求完全可以拖垮整个系统

### 5. 资料

* [NFS-RPC框架优化过程](http://bluedavy.me/?p=334)
* [BTrace简介及使用](http://blog.csdn.net/wildandfly/article/details/21107661)
* [Running BTrace custom script with GlassFish V3](https://blogs.oracle.com/nishigaya/entry/btrace_with_glassfish_v3)
