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
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-11-6/68911737.jpg)

- Startup/Connection,`Debug` Trasport-Socket - Port为远程调试端口
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-11-6/28951710.jpg)


# 三.端口查看与释放
- 确定进程(找到端口address=9996)
```
[admin@10-57-19-191 ~]$ ps -ef |grep 20974
admin 20974 1 2 15:41 ? 00:04:34 /usr/install/jdk1.8.0_60/bin/java -Djava.util.logging.config.file=/home/admin/billing/deploy/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -Xms2g -Xmx2g -XX:PermSize=512m -XX:SurvivorRatio=2 -XX:+UseParallelOldGC -Dtrace.flag=true -Dtrace.output.dir=/home/admin/billing/deploy/trace/ -server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=9996,server=y,suspend=n -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true -Dfile.encoding=UTF-8 -DdisableIntlRMIStatTask=true -Ddubbo.application.logger=slf4j -Djdk.tls.ephemeralDHKeySize=2048 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=10.57.19.191 -Djava.endorsed.dirs=/usr/install/tomcat/endorsed -classpath /usr/install/tomcat/bin/bootstrap.jar:/usr/install/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/home/admin/billing/deploy/tomcat -Dcatalina.home=/usr/install/tomcat -Djava.io.tmpdir=/home/admin/billing/deploy/tomcat/temp org.apache.catalina.startup.Bootstrap start
admin 24284 23346 0 18:18 pts/1 00:00:00 grep --color=auto 20974
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
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-11-6/37841573.jpg)
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-11-6/39263772.jpg)

## 2.本地字节码,同步远程服务器(`当前实例,重启失效`)
- 第一步: 先debug连接上远程服务器
- 第二步: 改代码,仅限方法内代码. 增加修改方法不可同步远程服务器.
- 第三步: Build Project,上次代码
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-11-6/37611496.jpg)

- 结果提示:
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-11-6/47208115.jpg)

