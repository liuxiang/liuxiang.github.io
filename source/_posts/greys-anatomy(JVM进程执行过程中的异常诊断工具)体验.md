title: greys-anatomy(JVM进程执行过程中的异常诊断工具)体验
date: 2018-8-16 00:00:02
categories: jvm
tags: [jvm]

---
[TOC]

---
# 概述
`Greys`是一个JVM进程执行过程中的异常诊断工具。 在不中断程序执行的情况下轻松完成JVM相关问题排查工作。
github: https://github.com/oldmanpushcart/greys-anatomy

---
## 功能
|命令|说明|
|--:|:--|
|[help](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#help)|查看命令的帮助文档，每个命令和参数都有很详细的说明|
|[sc](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#sc)|查看JVM已加载的类信息|
|[sm](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#sm)|查看已加载的方法信息|
|[monitor](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#monitor)|方法执行监控|
|[trace](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#trace)|渲染方法内部调用路径，并输出方法路径上的每个节点上耗时|
|[ptrace](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#ptrace)|强化版的`trace`命令。通过指定渲染路径，并可记录下路径中所有方法的入参、返值；与`tt`命令联动。|
|[watch](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#watch)|方法执行数据观测|
|[tt](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#tt)<br/>[详细用法](TimeTunnel)|方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测|
|[stack](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#stack)|输出当前方法被调用的调用路径|
|[version](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#version)|输出当前目标Java进程所加载的Greys版本号|
|[quit](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#quit)|退出greys客户端|
|[shutdown](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#shutdown)|关闭greys服务端|
|[reset](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#reset)|重置增强类，将被greys增强过的类全部还原|
|[jvm](https://github.com/oldmanpushcart/greys-anatomy/wiki/Commands#jvm)|查看当前JVM的信息|
完整见: `greys pdf`
https://github.com/oldmanpushcart/greys-anatomy/wiki/greys-pdf

---
# 示例
## `trace` 渲染方法内部调用路径，并输出方法路径上的每个节点上耗时
```xml
# 调用链路上的所有性能开销和追踪调用链路
ga?>trace *ClusterNodeCache* refresh
Press Ctrl+D to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 401 ms.
`---+Tracing for : thread_name="http-nio-8010-exec-6" thread_id=0x8f;is_daemon=true;priority=5;
    `---+[33,33ms]cn.***.watchdog.service.cache.ClusterNodeCache:refresh()
        +---[27,26ms]cn.***.watchdog.service.cache.ClusterNodeCache$ClusterService:getClusterServicesAll(@71)
        +---[30,1ms]java.util.Map:forEach(@72)
        +---[30,0ms]cn.***.watchdog.service.util.IpUtil:getIpAdd(@78)
        +---[30,0ms]java.lang.String:split(@78)
        `---[32,1ms]java.util.Map:forEach(@79)

# 为trace命令的强化版，通过指定渲染路径来完成对方法执行路径的渲染过程
ga?>ptrace *ClusterNodeCache* refresh
Press Ctrl+D to abort.
Affect(class-cnt:27 , method-cnt:94) cost in 1556 ms.
`---+pTracing for : thread_name="http-nio-8010-exec-4" thread_id=0x8d;is_daemon=true;priority=5;process=1024;
***
```
---
## `watch` 调试:查看入参,出参
```
# 入参
ga?>watch -b *ClusterNodeCache* getClusterServices '"params[0]="+params[0]'
Press Ctrl+D to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 371 ms.
params[0]=http://mock.t**d**.me:8080/api/v1/apps/ips?app_name=f**s***-api&env=production
或
ga?>watch -b *ClusterNodeCache* getClusterServices params -x 1
Press Ctrl+D to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 366 ms.
@Object[][
    @String[http://mock.t**d**.me:8080/api/v1/apps/ips?app_name=f**s***-api&env=production],
]

# 出参
ga?>watch -s *ClusterNodeCache* getClusterServices returnObj -x 1
Press Ctrl+D to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 423 ms.
@HashMap[
    @String[f**s***-pro1]:@HashSet[isTop=false;size=1],
    @String[f**s***-pro2]:@HashSet[isTop=false;size=1],
    @String[challenger_pro]:@HashSet[isTop=false;size=1],
]
@HashMap[
    @String[fg_pro3]:@HashSet[isTop=false;size=3],
    @String[f**s***-pro3]:@HashSet[isTop=false;size=3],
]
```
## `tt` 调用链录制
```
# 录制方法调用堆栈(支持条件录制)
ga?>tt -t -n 3 *ClusterNodeCache* getClusterServices
Press Ctrl+D to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 384 ms.
+----------+------------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
| INDEX | PROCESS-ID | TIMESTAMP | COST(ms) | IS-RET | IS-EXP | OBJECT | CLASS | METHOD |
+----------+------------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
| 1001 | 1095 | 2018-08-17 11:55:49 | 57 | true | false | 0x7e3cc3c3 | ClusterNodeCache$ClusterServic | getClusterServices |
| | | | | | | | e | |
+----------+------------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
| 1002 | 1097 | 2018-08-17 11:55:49 | 16 | true | false | 0x7e3cc3c3 | ClusterNodeCache$ClusterServic | getClusterServices |
| | | | | | | | e | |
+----------+------------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+
| 1003 | 1100 | 2018-08-17 11:55:50 | 16 | true | false | 0x7e3cc3c3 | ClusterNodeCache$ClusterServic | getClusterServices |
| | | | | | | | e | |
+----------+------------+----------------------+------------+----------+----------+-----------------+--------------------------------+--------------------------------+

# 展示录制列表
ga?>tt -l
(同上)

# 明细(堆栈,入参,出参)
ga?>tt -i 1002
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| INDEX | 1002 |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| PROCESS-ID | 1097 |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| GMT-CREATE | 2018-08-17 11:55:49 |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| COST(ms) | 16 |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| OBJECT | 0x7e3cc3c3 |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| CLASS | cn.***.watchdog.service.cache.ClusterNodeCache$ClusterService |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| METHOD | getClusterServices |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| IS-RETURN | true |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| IS-EXCEPTION | false |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| PARAMETERS[0] | http://mock.t**d**.me:8080/api/v1/apps/ips?app_name=f**s***-x&env=production |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| RETURN-OBJ | {fg_pro3=[10.57.19.175:8060, 10.57.19.175:8061, 10.57.19.175:8062], f**s***-pro3=[10.57.19.203:8013, 10.57.19.203:8012, 10.57.19.203:8011]} |
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| STACK | thread_name="http-nio-8010-exec-10" thread_id=0x93;is_daemon=true;priority=5; |
| | ***
+-----------------+--------------------------------------------------------------------------------------------------------------------------------------------------------+

# 重放
ga?>tt -i 1002 -p
```

## `stack` 目标方法有关的调用栈
```
ga?>stack *ClusterNodeCache* getClusterServices
Press Ctrl+D to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 378 ms.
thread_name="http-nio-8010-exec-2" thread_id=0x8b;is_daemon=true;priority=5;
    ***
```