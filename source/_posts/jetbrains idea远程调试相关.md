title: jetbrains idea远程调试相关
date: 2017-10-26 00:00:00
categories: idea
tags: [idea]

---

[TOC]

---
# 一.服务端配置:jvm
```
Jdk1.7之前: -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n 
jdk1.7之后: -agentlib:jdwp=transport=dt_socket,address=8000,server=y,suspend=n
```
# 二.客户端连接:idea远程调试配置及注意
- server处port非远程端口,`填错后果严重`
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/17-11-6/68911737.jpg)

- Startup/Connection,`Debug` Trasport-Socket - Port为远程调试端口
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/17-11-6/28951710.jpg)


# 三.端口查看与释放
- 确定进程(找到端口address=9996)
```
[admin@10-57-19-191 ~]$ ps -ef |grep 20974
```
- 查看连接发起方
```
[admin@10-57-19-191 ~]$ netstat -an|grep 9996
tcp 0 0 10.57.19.191:9996 10.57.240.227:56516 ESTABLISHED 20974/java 
```
```
# 端口服务进程
[admin@10-57-19-191 ~]$ ps -ef|grep 25323
admin 25323 1 7 18:37 ? 00:01:26 /usr/install/jdk1.8.0_60/
```
- 本地:查看连接目标
```
➜ ~ netstat -an|grep 9996
tcp4 0 0 10.57.240.227.58302 10.57.19.191.9996 ESTABLISHED
```
```
# 连接目标端口进程(`主动关闭此进程,可释放远程端口的占用`)
➜ ~ lsof -i :9996
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
idea 555 liuxiang 151u IPv4 0xe8ce82f432b2fc1f 0t0 TCP 10.57.240.227:58302->localhost:palace-5 (ESTABLISHED)
```

# 四.动态更新远程服务器
## 1.变量赋值Evaluate Expression (`当前线程栈有效,新线程失效`)
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/17-11-6/37841573.jpg)
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/17-11-7/22897891.jpg)

## 2.本地字节码,同步远程服务器(`当前实例,重启失效`)
- 第一步: 先debug连接上远程服务器
- 第二步: 改代码,仅限方法内代码. 增加修改方法不可同步远程服务器.
- 第三步: Build Project,上次代码
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/17-11-6/37611496.jpg)

- 结果提示:
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/17-11-6/47208115.jpg)

