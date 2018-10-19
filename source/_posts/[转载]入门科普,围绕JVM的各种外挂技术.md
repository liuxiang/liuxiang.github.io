title: 转载-入门科普,围绕JVM的各种外挂技术
date: 2018-8-25 00:00:02

---

[TOC]


---

`[转载]`http://calvin1978.blogcn.com/articles/vjtools-tools4.html


---

jstat, jmap, btrace, jprofiler, vjtools都基于什么实现？ 对围绕JVM的各种工具的外挂技术，运用大整理术，让大家从茫然，到轻摇纸扇，知道分子。
归拢一下，就是C 和 Java两种Agent，SA 和 VirtualMachine 两种 Attach，JMX和PerfData两种Data，两两之间很是混淆，网上好像少了篇简明扼要的科普，所以自己操刀补了一篇，作为上周的直播《VJTools如何利用佛性技术完全JVM》的续集。

# 1. 两种`Agent`

## 1.1 Native Agent
以 `C/C++代码编写的Agent`，用强大的`JVMTI（JVM Tool Interface）`接口与JVM进行通讯，订阅感兴趣的JVM事件（比如方法出入、线程始末等等），当这些事件发生时，会回调Agent的代码。 JVMTI 同时提供了众多的功能函数，查询和控制 Java 应用的运行状态，包括内存控制和对象获取，线程与锁等等，简直无所不能。
使用者，包括各种Profile工具，如`Yourkit，JProfiler，Aysnc-Profiler`, 还有动态Reload Class而不重启应用的`JRebel`。
使用方式 ，可以在`启动命令里加入 -agentlib: 或 -agentpath: /path/to/agent.so`，也可以用后面讲的VituralMachine.attach()动态加载。

## 1.2 Java Agent
Java Agent的底层也是`JVMTI `，但后门能力就只剩一个AOP 代码植入了：在`加载class文件之前做拦截并对字节码做修改`。比如`AspectJ`，单元测试覆盖率的`Jacoco`，动态重载Class的`Spring-Loaded`。
典型代码如下：

```
public class MyAgent {
    public static void premain(String args, Instrumentation instrumentation){
        ClassFileTransformer transformer = new MyClassWeaving();
        instrumentation.addTransformer(transformer);
    }
}

```
另一种逍遥的用法，就是为了随时让 “一段代码” 与 “主应用” 在同一JVM中运行，而不用修改主应用的代码去显式调用。
比如`JMX的agent`，启动一条TCP侦听线程响应JMX 请求。
比如`jolokia的agent`，启动一条Http侦听线程，响应Restful版的JMX请求。
比如`btrace`，启动一条TCP线程与btrace client通信，接收client发过来的脚本字节码，进行加载并输出结果。
它有两种启动方式：
- 一种是在启动时命令行加入 `-javaagent:/path/to/agent.jar`，根据agent.jar中的MANIFEST.MF文件中的Premain-Class定义，JVM找到相应的MyAgent类，调用其premain函数。
- 一种是通过后面讲的`VM.attach()`技术，在`任意时刻由外部程序来灵活加载`，调用其`agentmain`函数。

---
# 2.两种`Attach`
两种截然不同方式实现的Attach，本质上都是在`跟踪程序与目标JVM之间建立一个沟通的管道`，然后在跟踪程序使用特定的`API去操作目标JVM`。


## 2.1 Vitural Machine.attach()
跟踪程序通过`Unix Domain Socket `与目标`JVM的Attach Listener线程`进行交互。 Socket 文件为`/tmp/.java_pid$PID`。
API 接口是com.sun.tools.attach.VirtualMachine 及其子类`sun.tools.attach.HotSpotVirtualMachine`，在tools.jar中，运行时需要依赖，可以做下面的事情:
● dumpHeap： jmap -dump 效果
● heapHisto： jmap -histo效果
● threadDump： jstack效果
● dataDump： kill-3 效果(jstack + jmap -heap)
● loadAgent： 动态加载C/Java Agent
● agentProperties： 获得已加载Agent的属性
● sytemProperties： 获得System Properties
● setFlag： 动态设置可写的JVM参数（但没几个是可写的）
● printFlag： 打印JVM 参数的值
● jcmd： 执行jcmd命令，具体能干啥见jcmd $PID help
可见， jmap, jmap，jcmd 们默认就是基于这个机制来做事情的。
VJTools的vjmxcli的特色之一，如果应用的启动脚本忘了设定启动JMX，可以根据PID attach到应用里，动态的把JMX Agent启动起来，然后通Agent属性获得本地连接地址，嗯， jconsole也是这么干的。

## 2.2 SA.attach()
著名的`SA（Serviceability Agent）`，用于`分析JVM运行时进程的Snapshot数据`。Snapshot的意思，就是当SA 开始分析时，整个`目标JVM是停顿下来不工作`的，让SA可以从容读取进程内存中的数据，直到断开后才会恢复。所以在生产上使用这类工具时，`必须先摘除流量`。
这个神奇的操作，主要是通过系统调用ptrace实现。ptrace会使内核暂停目标进程并将控制权交给跟踪进程，使跟踪进程得以察看目标进程的内存，详见ptrace的man，所以在容器环境下，需要打开ptrace的安全权限。
API的接口，一个是 sun.jvm.hotspot.HotSpotAgent 负责attach， `sun.jvm.hotspot.runtime.VM`负责操作，在sa-jdi.jar中。

VM类从内存二进制信息中，提取出JVM内部数据结构，包括：
● 内存的getObjectHeap()/getUniverse()
● 处线程的getThreads()
● 永久代内容的getSymbolTable()，getStringTable()， getSystemDictionary()
● 还有很厉害的读内部Native对象值的getTypeDataBase()

jstack，jmap 们默认用前面的VM.attach()模式，与目标JVM的Attach Listener线程通信， 但如果目标JVM已经半死不活，Attach Listener线程无力响应请求时，就可以增加-F 参数，转而使用SA.attach 模式，用ptrace去暴力接管进程， 详细代码见sun.tools.jmap.JMap。看代码你还会发现，因为AttachListener支持的命令有限，所以jmap -heap 打印heap的总结信息时，也是以SA模式进去。


VJTools的vjmap是jmap的一个增强，能够独立各个分代中对象的数量大小统计，用以排除新生代对象的干扰，查找内存缓慢泄漏的问题，比如更直接找出Survior区里age>2的对象，里面就使用了VM类的观察者模式回调，和直接内存访问 两种方法来遍历。

`SA模式比VM模式`做相同事情时`要慢`一截，非必需时不要用它。还有，如果跟踪程序被kill－9 非正常退出，没有执行中断SA，目标JVM就会一直暂停在那里，Linux下可以执行kill -18 $PID 发送SIGCONT信号重新激活目标进程。



---
# 3. 两种Data
## 3.1 JMX
文章已太多，不再啰嗦。其中vjtools的vjtop，如何不停顿JVM的获得线程的CPU、内存信息，获得某条繁忙进程的StackTrace看看它在忙些什么，值得一看。


## 3.2 PerfData
很多人不知道的一个机制，JVM其实每秒都会将自己的大量统计数据，写入到 `/tmp/hsperfdata_$username/$pid` 文件中。
用下面指令可以感受下：

```
 jcmd $PID PerfCounter.print

```
内容包括`jvm的基本信息，内存，GC，线程数等等`，还有一些`JMX中没有暴露的数据`，比如包含JVM中所有的`停顿的SafePoint`信息。

```
//线程的情况
java.threads.daemon=6
java.threads.live=7
...
// younggen的情况
sun.gc.generation.0.capacity=44695552
sun.gc.generation.0.maxCapacity=715784192
sun.gc.generation.0.minCapacity=44695552
....

```
`jps，其实就是读取/tmp/hsperfdata_$username/ 目录下所有的文件`。
`jstat，同样是读取这个神秘的文件`。一个很大的好处，就是它只`默默读取文件，而不会像JMX那样要与应用程序交互，打扰应用程序的工作`。
自己写代码也简单，使用 `sun.management.counter.perf.PerfInstrumentation`类即可，VJTools里的vjtop 从`JMX，PerfData和/proc/pid` 三处地方，综合打印JVM的概况和繁忙线程的情况，里面的PerfData类就是其使用的展示。
`PerfData文件是mmap到内存中的，读写都很快`，但每次写完还要更新磁盘上的文件元数据比如last modified time，如果遇上磁盘高IO，还是有概率造成JVM被锁定一段时间。所以我们以前通过`-XX:+PerfDisableSharedMem禁止了perfdata的写入`，不过现在又有点摇摆。
注意，perfdata 和 vm.attach 都需要在/tmp 目录读写文件，如果目标JVM的启动参数重新指定了临时目录，而跟踪程序依然去读取/tmp 目录，也会导致这些机制失效。