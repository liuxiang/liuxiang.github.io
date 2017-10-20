title: jetbrains idea效率
date: 2017-7-04 00:00:00
categories: idea
tags: [JetBrains, IntelliJ IDEA]

---

[TOC]

# 一.常用键或功能
- 快捷键
    - `double Shift`: 万能(目标)搜索
    - `⌘ + Shift + F`: 全文搜索&实时预览
    - `⌘ + ,`：打开设置
    - `⌘ + Shift + A` : 软件功能搜索
    - `alt + F7` find usages (对象使用)
    - `⌘ + ←` / `⌘ + →`  回到上一次光标的代码位置/下一次光标位置
    - `control + Enter` 创建一切:新建文件,覆盖方法,get/set
    - `Alt + Enter` 修复警告:导包,修饰..
    - `Ctrl + Alt + L` 代码格式化
- 代码模板: `sout,psvm,iter,itar`
- plugins: `inspections/intentions`-[《阿里巴巴Java开发手册》扫描插件正式发布](https://yq.aliyun.com/articles/224324)
- 调试赋值: `Evaluate Expression [alt+F8]`
- idea database客户端
    - 本地sql存档:运维效率

**供参考**
`两张图让你完全了解IDEA和Android Studio所有快捷键 - 简书`
http://www.jianshu.com/p/8fc8d934ad19?utm_source=tuicool&utm_medium=referral

---
# 二.更多功能
## Java方法调用树
- Navigate | `Call Hierarchy` (调用层)（Hierarchy class  [ClassName]）[`Ctrl + H`]
![](http://images2015.cnblogs.com/blog/120296/201604/120296-20160412140642598-440700557.png)

## 对象引用 / 表达式、变量和方法参数等`数据的传递关系树`
> Analyze | `Dataflow from to Here`(数据流从这里到这里)
![](http://images2015.cnblogs.com/blog/120296/201604/120296-20160412140645613-1933987277.png)

`IDEA的查询引用、调用关系图的功能 - 华行天下 - 博客园`
http://www.cnblogs.com/huaxingtianxia/p/5728847.html

`IDEA的查询引用、调用关系图的功能 - 蝈蝈俊 - 博客园`
http://www.cnblogs.com/ghj1976/p/5382455.html

---
## 使用`Alt + Shift + C`快速查看您最近对项目的更改
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-10-20/75320506.jpg)

## Analyze This Dependency（分析Jar依赖关系）
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-10-20/97155812.jpg)
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-10-20/2019988.jpg)

## 查看Mavan工程的jar依赖层关系【`Show Dependencies（显示依赖关系）`】
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-10-20/44357164.jpg)

- 可以确认，jar包在maven间的应用关系
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-10-20/96690575.jpg)

## Show Diagram(uml模型图) 类的衍生关系(继承&实现)
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-10-20/99971312.jpg)
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-10-20/45708094.jpg)

---
# 三.零星功能
## 快速查看代码行作者
[侧边空白:右键-Annotate]
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-10-20/43683557.jpg)
或 [代码区:右键-git-Annotate]
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-10-20/88153192.jpg)

## [win]`Terminal`窗口中按快捷键 `F7` 可选择命令历史
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-10-20/63432870.jpg)

