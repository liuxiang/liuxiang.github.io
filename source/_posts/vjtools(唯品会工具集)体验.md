title: vjtools(唯品会工具集)体验
date: 2018-8-16 00:00:00
categories: jvm
tags: [jvm]

---
[TOC]

---

# 概述 

唯品会的java工具箱

`The vip.com's java coding standard, libraries and tools`
https://github.com/vipshop/vjtools

---

## Java Core Library
| Project | Description |
| -------- | -------- |
| [vjkit](https://github.com/vipshop/vjtools/tree/master/vjkit) | 关于文本，集合，并发等基础功能的核心类库 |
| [vjstar](https://github.com/vipshop/vjtools/tree/master/vjstar) | 关于后端应用的性能、可用性的最佳实践 |
## Java Tools
| Project | Description | Manual |
| -------- | -------- | -------- |
| [vjtop](https://github.com/vipshop/vjtools/tree/master/vjtop) | 观察JVM进程指标及其繁忙线程 | [Chinese](/vjtop/README.md), [English](/vjtop/README_EN.md)|
| [vjmap](https://github.com/vipshop/vjtools/tree/master/vjmap) | JMAP的分代打印版 |[Chinese](/vjmap/README.md), [English](/vjmap/README_EN.md)|
| [vjdump](https://github.com/vipshop/vjtools/tree/master/vjdump) | 线上紧急收集JVM数据脚本 | [Chinese](/vjdump/README.md), [English](/vjdump/README_EN.md)|
| [vjmxcli](https://github.com/vipshop/vjtools/tree/master/vjmxcli) | JMX 查看工具 | [Chinese](/vjmxcli/README.md)|


---

# 摘要
## `vjkit` 关于文本，集合，并发等基础功能的核心类库
- https://github.com/vipshop/vjtools/tree/master/vjkit
```
一是对Guava 与Common Lang中最常用的API的提炼归类，避免了大家直面茫茫多的API(但有些工具类如Guava Cache还是建议直接使用，详见直用三方工具类 )
二是对各门各派的精华的借鉴移植：比如一些大项目的附送基础库： Netty，ElasticSearch， 一些专业的基础库 ： Jodd, commons-io, commons-collections； 一些大厂的基础库：Facebook JCommon，twitter commons
```

- Usage

```

<dependency>
    <groupId>com.vip.vjtools</groupId>
    <artifactId>vjkit</artifactId>
    <version>1.0.5</version>
</dependency>

```

---
## `vjstar` 关于后端应用的性能、可用性的最佳实践
```
# JVM启动参数
参数兼顾性能及排查问题的便捷性的JVM启动参数推荐， 其中一些参数需要根据JDK版本适配。
https://github.com/vipshop/vjtools/blob/master/vjstar/src/main/script/jvm-options/jvm-options.sh
# 闲时主动GC
CMS GC 始终对流量有一定的影响。
因此我们希望在夜半闲时，如果检测到老生代已经达到50%， 则主动进行一次GC。
简单的定时器让应用固定在可设定的闲时（如半夜）进行清理动作。 为了避免服务的所有实例同时清理造成服务不可用，加入了随机值。
详见Proactive GC: https://github.com/vipshop/vjtools/tree/master/vjstar/src/main/java/com/vip/vjstar/gc
```

- Usage

```

<dependency>
    <groupId>com.vip.vjtools</groupId>
     <artifactId>vjstar</artifactId>
    <version>1.0.5</version>
</dependency>

```


---
## `vjtop` 打印JVM概况及繁忙线程工具

- Usage
```
<dependency>
    <groupId>com.vip.vjtools</groupId>
     <artifactId>vjtop</artifactId>
    <version>1.0.5</version>
</dependency>

Download 1.0.5.zip (from Maven Central)
http://repo1.maven.org/maven2/com/vip/vjtools/vjtop/1.0.5/vjtop-1.0.5.zip
```
- 示例
```
若你习惯以Top观察“OS指标及繁忙的进程”，也推荐以VJTop观看 “JVM进程指标 及 CPU最繁忙，内存占用最多的线程”。


# 2.3 找出`CPU`最繁忙的线程
// 按时间区间内，线程占用的CPU排序，默认显示前10的线程，默认每10秒打印一次
./vjtop.sh <PID>
// 按线程从启动以来的总占用CPU来排序
./vjtop.sh --totalcpu <PID>
// 按时间区间内，线程占用的SYS CPU排序
./vjtop.sh --syscpu <PID>
// 按线程从启动以来的总SYS CPU排序
./vjtop.sh --totalsyscpu <PID>


## 示例
 PID: 191082 - 17:43:12 JVM: 1.7.0_79 USER: calvin UPTIME: 2d02h
 PROCESS: 685.00% cpu(28.54% of 24 core), 787 thread
 MEMORY: 6626m rss, 6711m peak, 0m swap | DISK: 0B read, 13mB write
 THREAD: 756 active, 749 daemon, 1212 peak, 0 new | CLASS: 15176 loaded, 161 unloaded, 0 new
 HEAP: 630m/1638m eden, 5m/204m sur, 339m/2048m old
 NON-HEAP: 80m/256m/512m perm, 13m/13m/240m codeCache
 OFF-HEAP: 0m/0m direct(max=2048m), 0m/0m map(count=0), 756m threadStack
 GC: 6/66ms/11ms ygc, 0/0ms fgc | SAFE-POINT: 6 count, 66ms time, 5ms syncTime
    TID NAME STATE CPU SYSCPU TOTAL TOLSYS
     23 AsyncAppender-Worker-ACCESSFILE-ASYNC WAITING 23.56% 6.68% 2.73% 0.72%
    560 OSP-Server-Worker-4-5 RUNNABLE 22.58% 10.67% 1.08% 0.48%
   9218 OSP-Server-Worker-4-14 RUNNABLE 22.37% 11.45% 0.84% 0.40%
   8290 OSP-Server-Worker-4-10 RUNNABLE 22.36% 11.24% 0.88% 0.41%
   8425 OSP-Server-Worker-4-12 RUNNABLE 22.24% 10.72% 0.98% 0.47%
   8132 OSP-Server-Worker-4-9 RUNNABLE 22.00% 10.68% 0.90% 0.42%
   8291 OSP-Server-Worker-4-11 RUNNABLE 21.80% 10.09% 0.89% 0.41%
   8131 OSP-Server-Worker-4-8 RUNNABLE 21.68% 9.77% 0.93% 0.44%
   9219 OSP-Server-Worker-4-15 RUNNABLE 21.56% 10.43% 0.90% 0.41%
   8426 OSP-Server-Worker-4-13 RUNNABLE 21.35% 10.42% 0.66% 0.31%
 Total cpu: 668.56%(user=473.25%, sys=195.31%), 526 threads have min value
 Setting : top 10 threads order by CPU, flush every 10s
 Input command (h for help):



---
# 2.4 找出`内存`分配最频繁的线程
// 线程分配内存的速度排序，默认显示前10的线程，默认每10秒打印一次
./vjtop.sh --memory <PID>
// 按线程的总内存分配而不是打印间隔内的内存分配来排序
./vjtop.sh --totalmemory <PID>


## 示例
 PID: 191082 - 17:43:12 JVM: 1.7.0_79 USER: calvin UPTIME: 2d02h
 PROCESS: 685.00% cpu(28.54% of 24 core), 787 thread
 MEMORY: 6626m rss, 6711m peak, 0m swap | DISK: 0B read, 13mB write
 THREAD: 756 active, 749 daemon, 1212 peak, 0 new | CLASS: 15176 loaded, 161 unloaded, 0 new
 HEAP: 630m/1638m eden, 5m/204m sur, 339m/2048m old
 NON-HEAP: 80m/256m/512m perm, 13m/13m/240m codeCache
 OFF-HEAP: 0m/0m direct(max=2048m), 0m/0m map(count=0), 756m threadStack
 GC: 6/66ms/11ms ygc, 0/0ms fgc | SAFE-POINT: 6 count, 66ms time, 5ms syncTime
    TID NAME STATE CPU SYSCPU TOTAL TOLSYS
     23 AsyncAppender-Worker-ACCESSFILE-ASYNC WAITING 23.56% 6.68% 2.73% 0.72%
    560 OSP-Server-Worker-4-5 RUNNABLE 22.58% 10.67% 1.08% 0.48%
   9218 OSP-Server-Worker-4-14 RUNNABLE 22.37% 11.45% 0.84% 0.40%
   8290 OSP-Server-Worker-4-10 RUNNABLE 22.36% 11.24% 0.88% 0.41%
   8425 OSP-Server-Worker-4-12 RUNNABLE 22.24% 10.72% 0.98% 0.47%
   8132 OSP-Server-Worker-4-9 RUNNABLE 22.00% 10.68% 0.90% 0.42%
   8291 OSP-Server-Worker-4-11 RUNNABLE 21.80% 10.09% 0.89% 0.41%
   8131 OSP-Server-Worker-4-8 RUNNABLE 21.68% 9.77% 0.93% 0.44%
   9219 OSP-Server-Worker-4-15 RUNNABLE 21.56% 10.43% 0.90% 0.41%
   8426 OSP-Server-Worker-4-13 RUNNABLE 21.35% 10.42% 0.66% 0.31%
 Total cpu: 668.56%(user=473.25%, sys=195.31%), 526 threads have min value
 Setting : top 10 threads order by CPU, flush every 10s
 Input command (h for help):
```



## VJDump是线上JVM数据紧急收集脚本

- Usage

```

https://raw.githubusercontent.com/vipshop/vjtools/master/vjdump/vjdump.sh

```

- 示例

```

VJDump是线上JVM数据紧急收集脚本。
它可以在紧急场景下（比如马上要对进程进行重启），一键收集jstack、jmap以及GC日志等相关信息，并以zip包保存(默认在目录/tmp/vjtools/vjdump下)，保证在紧急情况下仍能收集足够的问题排查信息，减轻运维团队的工作量，以及与开发团队的沟通成本。
收集数据包括：
thread dump数据：jstack -l $PID
vjtop JVM概况及繁忙线程：vjtop.sh -n 1 $PID (需要将vjtop.sh 加入用户的PATH变量中)
jmap histo 堆对象统计数据：jmap -histo $PID & jmap -histo:live $PID
GC日志(如果JVM有设定GC日志输出)
heap dump数据（需指定--liveheap开启）：jmap -dump:live,format=b,file=${DUMP_FILE} $PID

# 对指定的进程PID进行急诊
vjdump.sh $pid

# 额外收集heap dump信息（jmap -dump:live的信息）
vjdump.sh --liveheap $pid

在收集过程中，某些命令如jmap -histo:live $PID 会造成JVM停顿，因此仅用于紧急情况或已摘流量的情况。为了避免连续停顿，在每条会造成停顿的收集指令之间，默认插入了1秒的执行间隔。

```

---
# 常用工具
```
# BTrace 系
btrace
greys 交互式免脚本，比btrace更易用
Java神器Btrace，从入门到熟练小工手册

# 在线日志分析
easygc.io gc日志分析
fastthread.io thread dump分析

# HeapDump分析
Eclipse MAT

# 性能Profile
Java Mission Control JDK自带Profiler
async-profiler 火焰图生成工具
```

---
**其它相关**
`关键业务系统的JVM参数推荐(2018仲夏版)`
http://calvin1978.blogcn.com/articles/jvmoption-7.html
`入门科普，围绕JVM的各种外挂技术`
http://calvin1978.blogcn.com/articles/vjtools-tools4.html