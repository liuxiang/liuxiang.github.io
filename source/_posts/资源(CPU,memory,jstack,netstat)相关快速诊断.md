title: 资源(CPU,memory,jstack,netstat)相关快速诊断
date: 2018-4-09 00:00:00
categories: 快速诊断
tags: [快速诊断]

---

[TOC]

---
# 一.java进程信息
```
# 查看java进程号(pid) : 
> jps
> pidof java

# 信息查看: 
> jinfo $pid
```

---
# 二.CPU (一键诊断: 进程下最消耗CPU的线程栈详情)
- 单行命令(压缩)
```
# 进程下最消耗CPU的线程栈详情
process=`pidof java`; echo $process;top1pid=`top -c -b -n 1 -Hp $process| sed -n '8,8p' | cut -d ' ' -f1`; thread=`printf '%x' $top1pid`;echo top1:$top1pid = $thread | grep top1;  [ -n $thread ] && jstack $process|grep 0x$thread -A15
```

- 分解
```
# 1.定位java进程
process=`pidof java`
echo $process "(如遇多个,自行赋值)"

# 2.定位top1线程pid(第八行:sed -n '8,8p')
top1pid=`top -c -b -n 1 -Hp $process| sed -n '8,8p' | cut -d ' ' -f1`

# 3.线程pid转十六进制
thread=`printf '%x' $top1pid`; echo $thread; 

# 4.打印top1线程栈[显示行数: 15 (可修改)]
[ -n $thread ] && jstack $process | grep 0x$thread -A15
```

- 提取线程ID的几种方式探索
```
echo `head + awk`
topT=`top -c -b -n 1 -Hp $process |grep admin|head -n 1| awk '$8>=$cpuload{print $1}'`

echo `cut -d`(空格分割,取数组位1)
topT=`top -c -b -n 1 -Hp $process |grep admin|head -n 1| cut -d ' ' -f1`

echo `tail -n +8` (第8行开始到最后)
topT=`top -c -b -n 1 -Hp $process|tail -n +8 | head -n 1 | cut -d ' ' -f1`

echo `sed -n '8,8p' ` (第8行开始到第8行)
topT=`top -c -b -n 1 -Hp $process|sed -n '8,8p' | cut -d ' ' -f1`
```

**参考**
`关于shell中，如何得到top命令显示的进程号？`
https://bbs.csdn.net/topics/350053680

---
# 三.内存
- GC情况：jstat -gc $pid 1000 5
- 堆使用情况：jmap -heap $pid
```
# 一键执行
pidof java | xargs jmap -heap
```

- 对象内存分布：jmap -histo:live $pid
```
# 一键执行
pidof java | xargs jmap -histo:live  | grep cn.fraudmetrix | head -n 20
```

- heap dump： jmap -dump:format=b,file=dumpfileName.dump $pid
```
pidof java | xargs jmap -dump:format=b,file=dumpfileName.dump 
```

---
# 四.线程堆栈(jstack)
```
# 打印堆栈
kill -3 [pid]
jstack [PID] >> jstack.log

# 显示系统java线程总数
ps -eLf | grep java -c
```

- [分组统计]堆栈中各线程状态
```
process=`pidof java`;jstack $process > jstack.log;
cat jstack.log | grep 'java.lang.Thread.State' | awk '{print $2" "$3" "$4" "$5}' | sort | uniq -c
# 效果
  34 RUNNABLE   
   4 TIMED_WAITING (on object monitor)
  11 TIMED_WAITING (parking)  
   7 TIMED_WAITING (sleeping)  
   2 WAITING (on object monitor)
  27 WAITING (parking)  
```

- [分组统计]各线程组
```
process=`pidof java`;jstack $process > jstack.log;
cat jstack.log | grep 'tid=' | awk '{print $1}' | awk -F '-' '{print $1"-"$2"-"$3}' | uniq -c | sort -rnk 1 | head -10
# 效果
  14 "http-nio-8010
   6 "New--
   4 "RMI--
   4 "GC--
   3 "JDWP--
   2 "metrics-meter-tick
   2 "C2--
   1 "watchdog_sync_data_liuxiangs-MacBook-Air.local
   1 "watchdog_sync_data_liuxiangs-MacBook-Air.local
   1 "statDailyScheduler-1"-
```

---
# 五.网络(连接情况)
```
# 连接数
netstat -na | wc -l

# 有效连接数
netstat -nat | grep ESTABLISHED | wc -l 

# 对各种状态的连接数分组统计结果
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'
SYN_SENT 2
LAST_ACK 1
CLOSE_WAIT 4
TIME_WAIT 256
ESTABLISHED 96
```
- 获取当前ip
```
inet_ip=`/sbin/ifconfig|grep inet|grep -v inet6|grep -v 127.0.0.1|awk '{if(substr($2,1,5)=="addr:"){print substr($2,6)} else{print $2}}'|head -n 1`
echo $inet_ip

或
inet_ip=`/sbin/ifconfig|grep inet|grep -v inet6|grep -v 127.0.0.1 | cut -d ' ' -f2`
echo $inet_ip

或
IP=`ip a|grep -w 'inet'|grep 'global'|sed 's/^.*inet //g'|sed 's/\/[0-9][0-9].*$//g'`
echo $IP
```

- 查看连接某服务端口最多的的IP地址
```
netstat -nat | grep "ESTABLISHED" | grep $inet_ip | awk '{print $5}' | awk -F: '{print $1}' | sort | uniq -c | sort -nr | head -20 
  14 192.168.6.28.80
  12 192.168.6.70.3306
  10 10.57.17.28.3306
   4 192.168.6.56.2181
   4 192.168.6.55.9092
   3 192.168.8.126.993
   2 192.168.6.57.2181
   2 192.168.6.56.9092
   2 151.101.72.133.443
   2 10.57.22.129.8080
   1 18.204.186.74.443
```

- curl获取http各阶段的响应时间
```
curl -o ~ -s -w %{time_namelookup}::%{time_connect}::%{time_starttransfer}::%{time_total}::%{speed_download}"\n" "baidu.com"
```

---
# 六.参考&工具
## tomcat jvm配置
```
# 服务端配置远程调试
- Jdk1.7之前:
CATALINA_OPTS="$CATALINA_OPTS -server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5888"
- jdk1.7之后:
CATALINA_OPTS="$CATALINA_OPTS -server -agentlib:jdwp=transport=dt_socket,address=8000,server=y,suspend=n"

# jconsole，VisualVM，JMC 监控配置（rmi）
# -Dcom.sun.management.jmxremote.authenticate=false 为 JMC 关闭飞行模式
JAVA_OPTS="$JAVA_OPTS -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.port=8999 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.managementote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Djava.rmi.server.hostname=***.***.***.***"
或
# set tomcat jmx
inet_ip=`/sbin/ifconfig|grep inet|grep -v inet6|grep -v 127.0.0.1|awk '{if(substr($2,1,5)=="addr:"){print substr($2,6)} else{print $2}}'|head -n 1`
CATALINA_OPTS="$CATALINA_OPTS -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=8999 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=$inet_ip"

# 配置tomcat，日常GC日志. 可记录下服务器历史的GC情况
# 注意：tomcat启动后如果webapps下未自动创建sys目录，请手动创建 */webapps>mkdir sys
# 配置路径到webapps下，方便日志文件的下载和直接定位：
# 访问：
# http://121.40.87.121:8080/sys/gc.log
# http://121.40.84.8:8080/sys/gc.log
# 可视化工具:GCViewer可以使用网络地址直接查看
CATALINA_OPTS="$CATALINA_OPTS -Xloggc:webapps/sys/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"

# jprofile服务器端配置
CATALINA_OPTS="$CATALINA_OPTS -agentlib:jprofilerti=port=8849,nowait -Xbootclasspath/a:/usr/local/jprofile/jprofiler9/bin/agent.jar"

# jvm shut down时,输出dump文件
CATALINA_OPTS="$CATALINA_OPTS -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/usr/local/apache-tomcat-7.0.59/logs/heap.dump"
```


---
## JDK工具箱 / 可视化工具集
### jvm内置(jconsole,VisualVM,JMC & JFR)
- jconsole
> ...

- VisualVM
> ...

- JMC & JFR
```
#pid=`jcmd | grep catalina | awk '{print $1}'`
pid=`pidof java`; 
jcmd $pid VM.unlock_commercial_features #先解锁技能

jcmd $pid JFR.start name=myrec settings=tongdun delay=20s duration=2m filename=/tmp/$pid.jfr
或
jcmd $pid JFR.start name=myrec delay=20s duration=2m filename=/tmp/$pid.jfr

#其中，delay参数表示profile延迟启动时间，duration表示持续采集时间，这里设置为2分钟
#settings表示使用哪种采集配置，这里用的就是第二步中放入的tongdun.jfc配置，它默认有一个名为profile的配置，如果不想采集异常信息，也可以直接用它。
jcmd $pid JFR.start name=myrec settings=tongdun delay=20s duration=2m filename=/tmp/$pid.jfr

#注意，采集数据生成后请执行下列命令移除这个采集
jcmd $pid JFR.stop name=myrec 
这样就好了，等待2分钟＋20秒，/tmp/pid.jfr文件就生成好了，这个文件直接导入JMC工具即可
```

### 三方(GCViewer,JProfiler)
- GCViewer
- 输出gc.log
```
CATALINA_OPTS="$CATALINA_OPTS -Xloggc:/usr/local/apache-tomcat-7.0.59/webapps/sys/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"
或(启动处)
CATALINA_OPTS="$CATALINA_OPTS -Xloggc:gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps"
```
- 访问 `open:open-file` 或 gc.log放置为静态资源:http://localhost:8080/gc.log

- JProfiler
> ...

- spring-boot-admin
https://github.com/codecentric/spring-boot-admin
![](https://github.com/codecentric/spring-boot-admin/raw/master/images/screenshot-details.png)

---
## dump分析(jvisualvm,IBM HeapAnalyzer)
- jvisualvm
```$xslt
生成dump:抽样器-内存-堆Dump (heapdump-***.hprof)
分析dump:文件-载入(下载服务器上dump:heapdump-***.hprof)
```
- IBM HeapAnalyzer(ha456.jar)
```
生成dump:jmap -dump:format=b,file=dumpfileName.dump $pid
分析dump:打开dumpfileName.dump
```

- 远程下载dump
```
scp admin@x.x.x.x:/home/admin/forseti-gateway/deploy/tomcat/temp/heapdump-1523203218367.hprof ~
```
