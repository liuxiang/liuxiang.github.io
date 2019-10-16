title: dubbo连接关系查看 
date: 2017-12-05 00:00:00 
categories: dubbo 
tags: [dubbo]

---
# 检查连接哪台机器的dubbo提供者(或哪些机器连接了本机的dubbo服务)
`netstat -an|grep 20880`  (再者可以根据dubbo-admin页面中查询)
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/18-1-15/78474250.jpg)

---
# 连接端口进程
```
➜  lsof -i :20880
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME
java 14225 liuxiang 63u IPv4 0xe82d742ed82d8037 0t0 TCP localhost:62197->localhost:20880 (ESTABLISHED)
java 14225 liuxiang 259u IPv4 0xe82d742ed4d0092f 0t0 TCP *:20880 (LISTEN)
java 14225 liuxiang 261u IPv4 0xe82d742ed391cb1f 0t0 TCP localhost:20880->localhost:65201 (ESTABLISHED)
➜lsof -i :20881
```