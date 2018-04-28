title: 火焰图（flame graph）是性能分析
date: 2018-2-5 00:00:00
categories: 火焰图
tags: [火焰图]

---
[TOC]

---
# 介绍
火焰图（flame graph）是性能分析的利器。
![](https://queue.acm.org/downloads/2016/Gregg4.svg)

## 火焰图生成方式
火焰图的生成过程有两步： 
1. 使用Profiler工具生成trace文件； 
2. 将trace文件转换为svg格式的火焰图文件；

---
# perf命令 [Profiler工具,采集]
让我们从 perf 命令（performance 的缩写）讲起，它是 Linux 系统原生提供的性能分析工具，会返回 CPU 正在执行的函数名以及调用栈（stack）
通常，它的执行频率是 99Hz（每秒99次），如果99次都返回同一个函数名，那就说明 CPU 这一秒钟都在执行同一个函数，可能存在性能问题。
```
$ sudo perf record -F 99 -p 13204 -g -- sleep 30
```
上面的代码中，perf record表示记录，-F 99表示每秒99次，-p 13204是进程号，即对哪个进程进行分析，-g表示记录调用栈，sleep 30则是持续30秒。
运行后会产生一个庞大的文本文件。如果一台服务器有16个 CPU，每秒抽样99次，持续30秒，就得到 47,520 个调用栈，长达几十万甚至上百万行。
为了便于阅读，perf record命令可以统计每个调用栈出现的百分比，然后从高到低排列。

---
# FlameGraph(制图工具)
包含堆栈跟踪的任何概要文件数据生成火焰图，包括从以下概要分析工具
(Flame graphs can be generated from any profile data that contains stack traces, including from the following profiling tools)
- Linux: `perf`, eBPF, SystemTap, and ktap
- Solaris, illumos, FreeBSD: DTrace
- Mac OS X: DTrace and Instruments
- Windows: Xperf.exe

http://www.brendangregg.com/flamegraphs.html
https://github.com/brendangregg/FlameGraph

---
# java Profiler工具
## 1.honest-profiler
有时候我们需要在CPU飙高的时候开始，这时候honest-profiler提供的动态启停功能就有用武之地了
启动成功后，通过telnet连接到127.0.0.1:12345，即可通过命令控制采样的`开始和结束`

## 2.lightweight-java-profiler
会从java虚拟机启动开始采样

## 3.更多采样工具
关于采样工具的选取，可以看看文章 Evaluating the Accuracy of Java Profilers，这里面列举了`xprof，hprof，jprofile和yourkit`四种采样器

`IDEA plugin` http://findtheflow.io/ 


## 4.`sjk` 查看堆栈或生成火焰图
```
java -jar sjk-0.9.jar ssa -f dump.std --print
java -jar sjk-0.9.jar ssa -f dump.std --histo
java -jar sjk-0.9.jar ssa -f dump.std --flame > flame.svg
```
![](https://img0.tuicool.com/AruQRbI.png!web)

`jvm排查工具箱jvm-tools`
https://www.tuicool.com/articles/7bMbYjY
https://github.com/aragozin/jvm-tools

`JAVA性能分析之使用火焰图 - CSDN博客`
http://blog.csdn.net/c395318621/article/details/55224665

`常用Java profiling工具的分析与比较`
http://blog.csdn.net/tanga842428/article/details/52403651

# Python profiler
## Visual profiler for Python
https://github.com/nvdv/vprof

---
**参考**
`如何读懂火焰图？ - 阮一峰的网络日志`
http://www.ruanyifeng.com/blog/2017/09/flame-graph.html