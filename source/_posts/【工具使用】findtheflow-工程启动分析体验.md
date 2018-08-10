title: 【工具使用】findtheflow-工程启动分析体验
date: 2018-1-10 00:00:02
tags: [findtheflow]

---
[TOC]

---
# 一.步骤
## 1.打开`Start recording from the beginning`
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-1-15/13523823.jpg)


## 2.启动(选择方式)
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-1-15/40465957.jpg)

## 3. 观察快照过程
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-1-15/43887016.jpg)

## 4.结束快照:手动stop (或calls超过50k会自动结束并保持快照)
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-1-15/44830451.jpg)

## 5.由于分析容量的限制(50K),会自动结束快照.(建议忽略部分packages参与统计)
```
The recording was truncated at 50K calls
This recording has generated a large number of calls so it has been truncated at 50K calls. We suggest that you shorten the recorded scenario or restrict the recording to more specific packages to reduce the number of recorded calls. Learn more >

The statistics below will help you identify the packages or classes that have generated the most calls. If some of them are not important for understanding your application, you can exclude them in the Run Configuration and record the same execution again.
```
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-1-15/66727011.jpg)

如图:忽略无关业务包`dal,util,common...` (目标尽量吧快照中调用量降低到50k内,这样能完成的统计出全过程)
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-1-15/42108678.jpg)

**(对于过大工程,在多重忽略package后,依然无法快照整个启动过程,即calls超过50k)**

---
# 二.效果
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-1-15/5770798.jpg)

增加忽略,延长快照:
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-1-15/81377537.jpg)

## 图解: `x轴:时间` `y轴:栈深度`
详见: http://findtheflow.io/docs/doc_intellij.html#_time_window

---
# 三.分析
1.第一部分,初始化`loadBypartners` 
2.第二部分,启动线程池`loadDecisionFlow`
3. ...(因calls超过50k,无法快照下来.)

