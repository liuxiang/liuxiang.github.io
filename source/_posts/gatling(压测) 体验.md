title: gatling(压测) 体验
date: 2018-4-05 00:00:00
categories: gatling
tags: [gatling]

---

[TOC]

---
# gatling 概述

---
# Simulation配置
- user-files/simulations/computerdatabase/BasicSimulation.scala
```
package computerdatabase // 1

import io.gatling.core.Predef._ // 2
import io.gatling.http.Predef._
import scala.concurrent.duration._

class BasicSimulation extends Simulation { // 3

  val httpConf = http // 4
    .baseURL("http://computer-database.gatling.io") // 5
    .acceptHeader("text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8") // 6
    .doNotTrackHeader("1")
    .acceptLanguageHeader("en-US,en;q=0.5")
    .acceptEncodingHeader("gzip, deflate")
    .userAgentHeader("Mozilla/5.0 (Windows NT 5.1; rv:31.0) Gecko/20100101 Firefox/31.0")

  val scn = scenario("BasicSimulation") // 7
    .exec(http("request_1") // 8
    .get("/")) // 9
    .pause(5) // 10

  setUp( // 11
    scn.inject(atOnceUsers(1)) // 12
  ).protocols(httpConf) // 13
}
```
```
1.可选包.
2.必要的需要import的包.
3.class 声明，继承自 Simulation.
4.所有HTTP请求的通用配置.
5.将被前置到所有相对URL的的baseUrl.
6.通用HTTP头信息，它将会被用于所有的requests请求中.
7.scenario 定义.
8.一个名叫 request_1的HTTP请求，请求名字将会显示在最终的报告中.
9.以 GET 方法请求目标 url.
10.一些 pause/think time.
11.设置将要在这个 Simulation 中启动的 scenarios.
12.在名叫 scn 的 scenario 中注入一个用户.
13.应用上面声明的 HTTP 配置.

备注：
val 是定义一个常量的关键字。变量类型没有定义，由Scala编译器确定.
时间单位默认为 seconds（秒），如：pause(5)等同于 pause(5 seconds).
关于Simulation 结构的详细信息，请查看 Simulation 参考页面.
```
https://gatling.io/docs/current/quickstart/#gatling-scenario-explained
https://testerhome.com/topics/3633

---
# 压测场景
## 1 几种压测场景示例：
```
//600秒跑1000个用户
setUp(scn.inject(rampUsers(100000) over (600 seconds)).protocols(httpConf))  

//10分钟内从每秒250个用户增长到每秒300个用户
setUp(scn.inject(rampUsersPerSec(250) to 300 during(10 minutes)).protocols(httpConf))

//一次并发每秒300用户
setUp(scn.inject(atOnceUsers(300)).protocols(httpConf))  
```

## 2 并发场景：
如果需要同时压多台机器，可以使用方法：
```
.baseURLs("http://10.0.0.1",“http://10.0.0.2")  
注：场景并发数据为压测多台机机并发数的总和
```

## 3 其他场景介绍：
```
setUp(  
  scn.inject(  
    nothingFor(4 seconds), // 1  
    atOnceUsers(10), // 2  
    rampUsers(10) over(5 seconds), // 3  
    constantUsersPerSec(20) during(15 seconds), // 4  
    constantUsersPerSec(20) during(15 seconds) randomized, // 5  
    rampUsersPerSec(10) to(20) during(10 minutes), // 6  
    rampUsersPerSec(10) to(20) during(10 minutes) randomized, // 7  
    splitUsers(1000) into(rampUsers(10) over(10 seconds)) separatedBy(10 seconds), // 8  
    splitUsers(1000) into(rampUsers(10) over(10 seconds)) separatedBy(atOnceUsers(30)), // 9  
    heavisideUsers(1000) over(20 seconds) // 10  
    ).protocols(httpConf)  
  )  
```
```
1、`nothingFor(4 seconds)`    在指定的时间段(4 seconds)内什么都不干
2、`atOnceUsers(10)`   一次模拟的用户数量(10)。
3、`rampUsers(10) over(5 seconds)`   在指定的时间段(5 seconds)内逐渐增加用户数到指定的数量(10)。
4、`constantUsersPerSec(10) during(20 seconds) `  以固定的速度模拟用户，指定每秒模拟的用户数(10)，指定模拟测试时间长度(20 seconds)。
5、`constantUsersPerSec(10) during(20 seconds) randomized`   以固定的速度模拟用户，指定每秒模拟的用户数(10)，指定模拟时间段(20 seconds)。用户数将在随机被随机模拟（毫秒级别）。
6、`rampUsersPerSec(10) to (20) during(20 seconds)`   在指定的时间(20 seconds)内，使每秒模拟的用户从数量1(10)逐渐增加到数量2(20)，速度匀速。
7、`rampUsersPerSec(10) to (20) during(20 seconds) randomized`   在指定的时间(20 seconds)内，使每秒模拟的用户从数量1(10)增加到数量2(20)，速度随机。
8、`splitUsers(10) into(rampUsers(10) over(10 seconds)) separatedBy(10 seconds)`    反复执行所定义的模拟步骤(rampUsers(100) over(10 seconds))，每次暂停指定的时间(10 seconds)，直到总数达到指定的数量(10)
9、`splitUsers(100) into(rampUsers(10) over(10 seconds)) separatedBy(atOnceUsers(30)) `  反复依次执行所定义的模拟步骤1(rampUsers(10) over(10 seconds))和模拟步骤2(atOnceUsers(30))，直到总数达到指定的数量(100)左右
10、`heavisideUsers(100) over(10 seconds)`    在指定的时间(10 seconds)内使用类似单位阶跃函数的方法逐渐增加模拟并发的用户，直到总数达到指定的数量(100).简单说就是每秒并发用户数递增。
```
`gatling详细使用 - CSDN博客`
https://blog.csdn.net/qq_37023538/article/details/54950827

# maven集成
- 配置
```
<properties>
    <gatling-charts-highcharts.version>2.3.1</gatling-charts-highcharts.version>
    <gatling-plugin.version>2.2.4</gatling-plugin.version>
</properties>

<dependencies>
    <dependency>
        <groupId>io.gatling.highcharts</groupId>
        <artifactId>gatling-charts-highcharts</artifactId>
        <version>${gatling-charts-highcharts.version}</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- gatling压测 -->
        <plugin>
            <groupId>io.gatling</groupId>
            <artifactId>gatling-maven-plugin</artifactId>
            <version>${gatling-plugin.version}</version>
            <configuration>
                <!-- 测试脚本 -->
                <simulationClass>c/test/scala</simulationClass>

                <!-- 结果输出地址 -->
                <resultsFolder>target/gatling/results</resultsFolder>

                <!-- 其它 -->
                <!--<configFolder>src/test/resources</configFolder>-->
                <!--<dataFolder>src/test/resources/data</dataFolder>-->
                <!--<bodiesFolder>src/test/resources/bodies</bodiesFolder>-->
            </configuration>
            <executions>
                <execution>
                    <id>execution-1</id>
                    <goals>
                        <goal>execute</goal>
                    </goals>
                    <configuration>
                        <simulationClass>scala/ForsetiGatewaySimulation</simulationClass>
                    </configuration>
                </execution>
                <!-- Here, can repeat the above execution segment to do another test -->
            </executions>
        </plugin>
    </plugins>
</build>
```

- 运行
```
mvn gatling:execute
```

`GETTING STARTED WITH SCALA IN INTELLIJ`
https://docs.scala-lang.org/getting-started-intellij-track/getting-started-with-scala-in-intellij.html

# 相关资源
- plugins.jetbrains scala 下载
https://plugins.jetbrains.com/plugin/1347-scala

# 问题
## object gatling is not a member of package io
```
10:57:26.167 [main][ERROR][ZincCompiler.scala:140] i.g.c.ZincCompiler$ - /Users/liuxiang/Desktop/work/forseti-zuul/forseti-gateway/service/src/test/scala/ForsetiGatewaySimulation.scala:18: object gatling is not a member of package io
10:57:26.179 [main][ERROR][ZincCompiler.scala:140] i.g.c.ZincCompiler$ - import io.gatling.core.Predef._
```
- 尝试一: 导入scala-SDK. (结果:未解决,`io.gatling.core.Predef._`非`scala`内容,而是`gatling`内容)

- 尝试二: 检查gatling依赖.结果:依赖完整,但无法在compile时不能识别.



---
**相关**
`Gatling Load and Performance testing - Open-source load and performance testing`
https://gatling.io/docs/current/quickstart/
翻译: https://testerhome.com/topics/3633

`性能测试工具——gatling - 乌鸦不会飞`
http://www.bigerhead.com/2016/11/330.html

`使用Gatling做web压力测试 - DTeam的团队日志 - SegmentFault 思否`
https://segmentfault.com/a/1190000008254640

`性能测试之 Gatling - 推酷`
https://www.tuicool.com/articles/fiemeyN

`jenkins：应用篇（Gatling plugin的使用） - shihuc - 博客园`
https://www.cnblogs.com/shihuc/p/5149035.html
https://www.bbsmax.com/A/QW5YXDnYJm/
