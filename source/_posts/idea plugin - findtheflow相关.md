title: idea plugin - findtheflow相关
date: 2018-1-10 00:00:00
categories: findtheflow
tags: [findtheflow]

---
[TOC]

---
# 一.介绍
## 1.简述
`findtheflow`记录您的应用程序执行情况，并通过交互式Web界面可视化运行时发生的事件

## 2.效果
![](http://findtheflow.io/docs/images/time_window_select_timeline.gif)
示例: http://app.findtheflow.io/?demo=true

## 3.安装
![](http://findtheflow.io/docs/images/install_plugin.png)

## 4.站点
http://findtheflow.io/
https://github.com/findtheflow/Feedback

## 5.文档
`Flow Documentation`
http://findtheflow.io/docs/doc_intellij.html

---
# 二.体验(localhost:7575)
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-1-15/52922894.jpg)
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-1-15/60089987.jpg)

## 1.Recordings 堆栈记录,可通过`Explore`观察堆栈视图.
- 场景1:junit单测程序,在结束时会自动生成.
- 场景2:`executions`主动`Record ~ Stop`自动生成`Recording`

## 2.executions: 常规正在运行时的应用程序(不支持实时的`Explore`,而是主动`Record ~ Stop`可自动生成`Recording`)
- 场景1:main函数测试类在运行后立即消亡,Executions中也会同时消失.如需保留`Recording`,需勾选flow-`Start recording from the beginning`
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-1-15/13523823.jpg)

`how_to_record_an_execution`
```
An execution is recorded automatically from the beginning if the Start recording from the beginning option is checked in the Run Configuration.
```
http://findtheflow.io/docs/doc_intellij.html#_how_to_record_an_execution

- 场景2:记录应用的启动堆栈.方法同`场景1`设置勾选`flow-Start recording from the beginning`
- 场景3:对运行时,片段执行程序生成堆栈视图. 在操作前点击`Record`,操作后点击`Stop`,会自动生成`Recording`

---
# 三.不使用IntelliJ？尝试流单机版 `How to use Flow standalone version?`
> We recommend you to use Flow in standalone mode only if you do not run your application from IntelliJ. You can skip this section if you are using Flow plugin for IntelliJ.

## 1.Download and Installation
Download the standalone version of Flow. Unzip it to any place you like.
http://download.findtheflow.io/flow-20171022095110.zip

## 2.Startup
Run Flow standalone version with command:
`java -jar flow-${version-number}.jar`
You will see the message below when Flow is started.
```
$ java -jar flow-20171022095110.jar
______ _
| ___|| |
| |_ | | ___ __ __
| _| | | / _ \ \ \ /\ / /
| | | || (_) | \ V V /
\_| |_| \___/ \_/\_/
Flow is starting...
Flow is ready.
```

## 3.Configure and run your application with Flow
You need to run your application with the following VM options:
- `-javaagent:${user-directory}/.flow/resources/javaagent.jar`
- `-Dflow.agent.include=com.foo` : packages and classes to include to the recording (comma-separated)
- `-Dflow.agent.exclude=com.foo,com.bar.Baz (optional)`: packages and classes to exclude from the recording (comma-separated)
- `-Dflow.agent.autostart (optional)`: record the application from the beginning
- `-Dflow.agent.execution-name=myapp (optional)`: execution and recording name displayed in the Flow webapp

详见: http://findtheflow.io/docs/doc_intellij.html#_how_to_use_flow_standalone_version

---
**更多参考** 
`分析代码调用关系的利器-Flow`
http://leaver.me/2017/04/08/%E5%88%86%E6%9E%90%E4%BB%A3%E7%A0%81%E8%B0%83%E7%94%A8%E5%85%B3%E7%B3%BB%E7%9A%84%E5%88%A9%E5%99%A8-Flow/?nsukey=ATLdNDD4Kz4x%2BsbQKUhSwZaDyGt%2B7Eg3iIGA89MmZXM7CklM9RqlodA5%2Fny%2BRr5RiO1iVW%2FkHRBKBYS4bBv3rtJlIkIyUkODQYPniO3mFbs%2BwebckwUGNS%2BmyMl%2BK%2BsMjkLmm%2BQoKlIamcNGVBcGTWNLQctDVF7yh2Ylav97oOm4amhFwiThyif2%2F7oJHUuvZuiNdzm8ugtHtjGpCQ29pw%3D%3D
