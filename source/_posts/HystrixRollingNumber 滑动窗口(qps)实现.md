title: HystrixRollingNumber 滑动窗口(qps)实现
date: 2018-10-14 00:00:00
categories: 限流
tags: [限流]


---

[TOC]



---

# HystrixRollingNumber 滑动窗口,使用示例

```

package com.netflix.hystrix.util;



import com.netflix.hystrix.util.HystrixRollingNumber;

import com.netflix.hystrix.util.HystrixRollingNumberEvent;



public class Main {



    /* package */

    static final Integer default_metricsRollingStatisticalWindow = 10000;// default => statisticalWindow: 10000 = 10 seconds (and default of 10 buckets so each bucket is 1 second)

    private static final Integer default_metricsRollingStatisticalWindowBuckets = 10;// default => statisticalWindowBuckets: 10 = 10 buckets in a 10 second window so each bucket is 1 second



    public static void main(String[] args) throws InterruptedException {

        HystrixRollingNumber counter

                = new HystrixRollingNumber(default_metricsRollingStatisticalWindow, default_metricsRollingStatisticalWindowBuckets);



        int i = 0;

        long timeBegin = System.currentTimeMillis();

        while (true) {

            counter.increment(HystrixRollingNumberEvent.SUCCESS);// 事件+增量



            if (i++ > 2000000) {

                System.out.println(counter.buckets.size()

                        + " : " + counter.getRollingSum(HystrixRollingNumberEvent.SUCCESS)

                        + " : " + counter.getRollingSum(HystrixRollingNumberEvent.SUCCESS) / counter.buckets.size()+"(qps)");

                i = 0;

                // Thread.sleep(100);

            }

        }



        /**

         * 分析结论:桶越多,qps提取越接近(精准)

         * 1 : 2000002 : 2000002(qps)

         * 1 : 4000004 : 4000004(qps)

         * 1 : 6000006 : 6000006(qps)

         * 1 : 8000008 : 8000008(qps)

         * 1 : 10000010 : 10000010(qps) -- 第一个桶(1s1桶),仅在最后时间的最终值才是准确的qps

         * 2 : 12000012 : 6000006(qps)

         * 2 : 14000014 : 7000007(qps)

         * 2 : 16000016 : 8000008(qps)

         * 2 : 18000018 : 9000009(qps)

         * 3 : 20000020 : 6666673(qps)

         * 3 : 22000022 : 7333340(qps)

         * 3 : 24000024 : 8000008(qps)

         * 3 : 26000026 : 8666675(qps)

         * 3 : 28000028 : 9333342(qps)

         * 3 : 30000030 : 10000010(qps)

         * 4 : 32000032 : 8000008(qps)

         * 4 : 34000034 : 8500008(qps)

         * 4 : 36000036 : 9000009(qps)

         * 4 : 38000038 : 9500009(qps)

         * 4 : 40000040 : 10000010(qps)

         * 4 : 42000042 : 10500010(qps)

         * 4 : 44000044 : 11000011(qps)

         * 5 : 46000046 : 9200009(qps)

         * 5 : 48000048 : 9600009(qps)

         * 5 : 50000050 : 10000010(qps)

         * 5 : 52000052 : 10400010(qps)

         * 5 : 54000054 : 10800010(qps)

         * 5 : 56000056 : 11200011(qps)

         * 6 : 58000058 : 9666676(qps)

         * 6 : 60000060 : 10000010(qps)

         * 6 : 62000062 : 10333343(qps)

         * 6 : 64000064 : 10666677(qps)

         * 6 : 66000066 : 11000011(qps)

         * 6 : 68000068 : 11333344(qps)

         * 6 : 70000070 : 11666678(qps)

         * 7 : 72000072 : 10285724(qps)-- 第7个桶时,已经可以在任意点提取qps,相对准确

         * 7 : 74000074 : 10571439(qps)

         * 7 : 76000076 : 10857153(qps)

         * 7 : 78000078 : 11142868(qps)

         * 7 : 80000080 : 11428582(qps)

         * 7 : 82000082 : 11714297(qps)

         * 8 : 84000084 : 10500010(qps)

         * 8 : 86000086 : 10750010(qps)

         * 8 : 88000088 : 11000011(qps)

         * 8 : 90000090 : 11250011(qps)

         * 8 : 92000092 : 11500011(qps)

         * 8 : 94000094 : 11750011(qps)

         * 9 : 96000096 : 10666677(qps)

         * 9 : 98000098 : 10888899(qps)

         * 9 : 100000100 : 11111122(qps)

         * 9 : 102000102 : 11333344(qps)

         * 9 : 104000104 : 11555567(qps)

         * 9 : 106000106 : 11777789(qps)

         * 10 : 108000108 : 10800010(qps)

         * 10 : 110000110 : 11000011(qps)

         * 10 : 112000112 : 11200011(qps)

         * 10 : 114000114 : 11400011(qps)

         * 10 : 116000116 : 11600011(qps)

         * 10 : 118000118 : 11800011(qps)

         * 10 : 109159233 : 10915923(qps)

         */

    }



}



/**

 * Hystrix/HystrixRollingNumberTest.java · Netflix/Hystrix

 * https://github.com/Netflix/Hystrix/blob/7f5a0afc23aa5ff82320560a04d4c81a45efd67c/hystrix-core/src/test/java/com/netflix/hystrix/util/HystrixRollingNumberTest.java

 */

```



---

# 结构

- `HystrixRollingNumber`: 滑动窗口实例类

- `BucketCircularArray`: 环形列表,内部由`ListState(环形)`存储`Buckets`,通过`buckets.peekLast()`获取尾端内容(Bucket)

- `ListState`: 环形存储(窗口数+1),其次使用`原子引用数组(AtomicReferenceArray<Bucket>)`存储桶

    - 其中`head,tail`来标记,开始桶位与结束桶位.

        - 在桶数未满时,head=0,tail=当前累计桶数

        - 当桶数饱和时,head=(head + 1) % dataLength[原head位+1],(tail + 1) % dataLength[原tail位+1]

        - `% dataLength`保障了`head,tail`数不会超过桶数,会循环轮转.

- `Bucket`: 单桶,有自己的`开始时间`,`生命周期(时长,如:1000ms)`/多单位`计数器(LongAdder`:利用此可统计不同状态的情况计数)


## 图示
- 窗口
![](https://github.com/Netflix/Hystrix/wiki/images/rolling-stats-640.png)
https://github.com/Netflix/Hystrix/wiki/Configuration#metrics

![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/18-10-18/79718668.jpg)
- 设计
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/18-10-18/83318069.jpg)
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/18-10-18/41615557.jpg)

---
# 程序设计
```
public void increment(HystrixRollingNumberEvent type) {
    getCurrentBucket().getAdder(type).increment();
}
```
- 初始化滑动窗口实例时,创建环形列表(ListState),其中无桶(Bucket)
- 请求进来,先获取末端桶,不存在时创建.初始化当前桶开始时间(System.currentTimeMillis()),桶区间(时间,如:1000ms),添加到环形列表(ListState)
    - 环形列表(ListState)中`head,tail`会标记,开始桶位与结束桶位.
        - 在桶数未满时,head=0,tail=当前累计桶数
        - 当桶数饱和时,head=(head + 1) % dataLength[原head位+1],(tail + 1) % dataLength[原tail位+1]
        - `% dataLength`保障了`head,tail`数不会超过桶数,会循环轮转.
- 后续请求进来,同样先获取末端桶,存在时比较`当前时间与桶区间`是否时间溢出
    - `不溢出`: 返回当前桶
    - `溢出值超过窗口时间`: 重新初始化环形列表(ListState)
    - `溢出值在窗口时间内,但相隔多个桶周期`: 循环桶次数中 新建桶,设置桶开始时间(末端桶时间),桶区间(时间,如:1000ms),直到时间合适的当前时间桶
- 获得末端桶后,操作其中的计数器(LongAdder)
    - LongAdder计数器,可按类型(单位)分别计数.所有类型总计数为桶的总计数

---
## 设计思路
> 需求: 在统计指标项时，如果每个周期都从零开始统计，那么会得到一个周期性出现锯齿的统计曲线. 
通过移动时间窗口淘汰最老的bucket,收集窗口时间内各bucket的计量,可推得平滑稳定的qps.

**特性:**
- 滑动窗口: 需要一个数组,每个位存储独立的计数
- 窗口可滑动: 按指定位存储.通过取模(%)控制循环写入位
- 窗口中每个位有生命周期: 抽象桶对象,为数组位对象.其中设置开始时间,生命时长,独立计数器
- 并发安全: 操作数组替换为原子引用数组(AtomicReferenceArray)
- 场景分支: 每个请求,默认计数到末端桶位.但请求存在停顿(请求及少)情况,需要位移多个桶才能找到合适计数位
    - 停顿时间>整个窗口(多桶): 重置窗口(所有桶)计数器
    - 停顿时间<整个窗口(多桶): 中间新间空桶,追赶到合适的桶位,开始计数

---
# 关键源码
- 初始化实例: new HystrixRollingNumber
```
    public HystrixRollingNumber(int timeInMilliseconds, int numberOfBuckets) {
        this(ACTUAL_TIME, timeInMilliseconds, numberOfBuckets);
    }
    /* package for testing */ HystrixRollingNumber(Time time, int timeInMilliseconds, int numberOfBuckets) {
        this.time = time;
        this.timeInMilliseconds = timeInMilliseconds;// 窗口时长
        this.numberOfBuckets = numberOfBuckets;// 分桶数
        if (timeInMilliseconds % numberOfBuckets != 0) {
            throw new IllegalArgumentException("The timeInMilliseconds must divide equally into numberOfBuckets. For example 1000/10 is ok, 1000/11 is not.");
        }
        this.bucketSizeInMillseconds = timeInMilliseconds / numberOfBuckets;
        buckets = new BucketCircularArray(numberOfBuckets); // 创建环形列表
    }
```
- 计数增一
```
    public void increment(HystrixRollingNumberEvent type) {
        getCurrentBucket().getAdder(type).increment();
    }
    
    // 获取桶
    /* package for testing */Bucket getCurrentBucket() {
        long currentTime = time.getCurrentTimeInMillis();
        Bucket currentBucket = buckets.peekLast();

        // 已存在,且有效
        if (currentBucket != null && currentTime < currentBucket.windowStart + this.bucketSizeInMillseconds) {
            return currentBucket;
        }

        if (newBucketLock.tryLock()) {
            try {
                // 首次不存在末端桶时,创建
                if (buckets.peekLast() == null) {
                    // the list is empty so create the first bucket
                    Bucket newBucket = new Bucket(currentTime);
                    buckets.addLast(newBucket);
                    return newBucket;
                } else {
                    // 存在桶,但无效,需创建(考虑超期多久)
                    for (int i = 0; i < numberOfBuckets; i++) {
                        // we have at least 1 bucket so retrieve it
                        Bucket lastBucket = buckets.peekLast();
                        if (currentTime < lastBucket.windowStart + this.bucketSizeInMillseconds) {
                            return lastBucket;
                        } else if (currentTime - (lastBucket.windowStart + this.bucketSizeInMillseconds) > timeInMilliseconds) {
                            reset();// 距离上次请求,超过了总窗口时间,计数器重置
                            return getCurrentBucket();
                        } else { 
                            // 距离上次请求,大于当前桶时间,小于总窗口时间.在此循环总桶数间追赶匹配的桶位
                            buckets.addLast(new Bucket(lastBucket.windowStart + this.bucketSizeInMillseconds));
                            cumulativeSum.addBucket(lastBucket);
                        }
                    }
                    return buckets.peekLast();
                }
            } finally {
                newBucketLock.unlock();
            }
        } else {
            currentBucket = buckets.peekLast();
            if (currentBucket != null) {
                return currentBucket;
            } else {
                try {
                    Thread.sleep(5);
                } catch (Exception e) {
                }
                return getCurrentBucket();
            }
        }
    }

        // 当前桶的按类型(LongAdder)计数
        LongAdder getAdder(HystrixRollingNumberEvent type) {
            if (!type.isCounter()) {
                throw new IllegalStateException("Type is not a Counter: " + type.name());
            }
            return adderForCounterType[type.ordinal()];
        }
```


---

**参考**

`浅析HystrixRollingNumber(用于qps计数的数据结构) - 简书`

https://www.jianshu.com/p/aca80fe37c86



`Hystrix学习笔记一 - marsflow的博客 - CSDN博客`

https://blog.csdn.net/marsflow/article/details/51604881



`Hystrix浅入浅出：（二）断路器和滑动窗口 - 大步流星的博客 - CSDN博客`

https://blog.csdn.net/manzhizhen/article/details/80296655