title: Rxjava,Rxjava2 相关
date: 2018-5-16 00:00:00
categories: Rxjava2
tags: [Rxjava2]

---

[TOC]

---
# 示例
- Observable,subscribe
```
Observable.create((e) -> {
    e.onNext(1);
    e.onNext(2);
})
        .doOnNext(System.out::println)
        .map((l) -> "map:" + l)
        .subscribe(System.out::println);
```
- Flowable/Observable
```
Flowable<Long> flowable =
        Flowable.create((FlowableOnSubscribe<Long>) e -> {
            Observable.interval(10, TimeUnit.MILLISECONDS)
                    .take(Integer.MAX_VALUE)
                    .subscribe(e::onNext);
        }, BackpressureStrategy.DROP);

Observable<Long> observable =
        Observable.create((ObservableOnSubscribe<Long>) e -> {
            Observable.interval(10, TimeUnit.MILLISECONDS)
                    .take(Integer.MAX_VALUE)
                    .subscribe(e::onNext);
        });
```

---
# 特性
- `RxJava / RxJava2`  (https://github.com/ReactiveX/RxJava)
    - https://github.com/ReactiveX/RxJava/wiki
        - How To Use RxJava(如何使用) https://github.com/ReactiveX/RxJava/wiki/How-To-Use-RxJava
        - What's different in 2.0 (在2.0有什么不同) https://github.com/ReactiveX/RxJava/wiki/What's-different-in-2.0
    - https://github.com/mcxiaoke/RxDocs  `中文doc`
    - https://github.com/kaushikgopal/RxJava-Android-Samples  `示例`
    - http://gank.io/post/560e15be2dca930e00da1083#toc_8 `示例`

## 观察者模式
- Observable ( 被观察者 ) / Observer ( 观察者 )
- Flowable （被观察者）/ Subscriber （观察者）
![](https://upload-images.jianshu.io/upload_images/3994917-21e4dcc1b5e3196a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

## RxJava2
`这可能是最好的RxJava 2.x 教程（完结版） - 简书` https://www.jianshu.com/p/0cd258eecf60
`给初学者的RxJava2.0教程(一)` https://www.jianshu.com/p/464fa025229e
`给初学者的RxJava2.0教程(二)` https://www.jianshu.com/p/8818b98c44e2
`可能是最好的 Rx 初学者教程` https://zhuanlan.zhihu.com/p/25552305

### RxJava2 vs RxJava1
- RxJava2最大的改动就是对于backpressure的处理，为此将原来的Observable拆分成了新的Observable和Flowable，同时其他相关部分也同时进行了拆分。https://www.jianshu.com/p/850af4f09b61
- rxjava1 到 rxjava2的一些类名和类的方法名发生了变化。因此在使用new方式来设置对象的时候，通过要把类名和方法名字一起改了，如果使用的是lambda表达式，那么基本不用改代码。[+变更事项]   https://blog.csdn.net/weixin_39595561/article/details/78463173
- RxJava1 跟 RxJava2 不能共存.[+变更事项]  https://www.jianshu.com/p/6d644ca1678f
- Subject是RxJava1.x中就有的，继承自Observable，所以不支持背压，Processor是RxJava2.x中新加入的，所以支持背压 [+变更事项]  https://blog.csdn.net/jeasonlzy/article/details/74269443
- RxJava2与RxJava的比较 https://blog.csdn.net/jianesrq0724/article/details/54892758

---
## Chaining Operators(操作符)
详见: http://reactivex.io/documentation/operators.html / https://github.com/mcxiaoke/RxDocs
- `Creating Observables 创建`
    - Create,Defer,Empty/Never/Throw,From,Interval,Just,Range,Repeat,Start,Timer
- `Transforming Observables 转换`
    - Buffer,FlatMap,GroupBy,Map,Scan,Window
- `Filtering Observables 过滤`
    - Debounce,Distinct,ElementAt,Filter,IgnoreElements,Last,Sample,Skip,SkipLast,Take,TakeLast
- `Combining Observables 组合`
    - And/Then/When,CombineLatest,Join,Merge,StartWith,Switch,Zip
- `Error Handling Operators 异常`
    - Catch,Retry
- `Observable Utility Operators 工具`
    - Delay,Do,Materialize/Dematerialize,ObserveOn,Serialize,Subscribe,SubscribeOn,TimeInterval,Timeout,Timestamp,Using,
- `Conditional and Boolean Operators 条件`
    - All,Amb,Contains,DefaultIfEmpty,SequenceEqual,SkipUntil,SkipWhile,TakeUntil,TakeWhile,
- `Mathematical and Aggregate Operators 计算`
    - Average,Concat,Count,Max,Min,Reduce,Sum
- `Backpressure Operators 背压`
    - backpressure operators
- `Connectable Observable Operators 连接`
    - Connect,Publish,RefCount,Replay,
- `Operators to Convert Observables 转换`
    - To
- `Async 异步操作`
    - Start/ToAsync/StartFuture/FromAction/FromCallable/RunAsync
- `Connect 连接操作`
    - Connect/Publish/RefCount/Replay
- `Convert 转换操作`
    - ToFuture/ToList/ToIterable/ToMap/toMultiMap
- `Blocking`
    - 阻塞操作 - ForEach/First/Last/MostRecent/Next/Single/Latest
- `String 字符串操作`
    - ByLine/Decode/Encode/From/Join/Split/StringConcat
- `操作符决策树模型`
    - http://reactivex.io/documentation/operators.html

---
## observeOn()与subscribeOn() 线程变换
`你真的会用RxJava么?RxJava线程变换之observeOn与subscribeOn - 简书`
https://www.jianshu.com/p/59c3d6bb6a6b

`RxJava之五—— observeOn()与subscribeOn()的详解`
https://blog.csdn.net/xx326664162/article/details/51967967

---
## Single - 一种特殊的只发射单个值的Observable
https://github.com/mcxiaoke/RxDocs/blob/master/Single.md

`这可能是最好的RxJava 2.x 入门教程（四） - 简书`
https://www.jianshu.com/p/c08bfc58f4b6

---
## Subject - Observable和Observer的复合体，也是二者的桥梁
https://github.com/mcxiaoke/RxDocs/blob/master/Subject.md

---
## 调度器(Schedulers) - 介绍了各种异步任务调度和默认调度器
`RxJava 第三篇 - Scheduler调度器使用及示例 - 简书`
https://www.jianshu.com/p/b037dbae9d8f

`RxJava之调度器(Schedulers) - CSDN博客`
https://blog.csdn.net/io_field/article/details/51429519



---
# 场景
简单的网络请求
- 1）通过 Observable.create() 方法，调用 OkHttp 网络请求；
- 2）通过 map 操作符集合 gson，将 Response 转换为 bean 类；
- 3）通过 doOnNext() 方法，解析 bean 中的数据，并进行数据库存储等操作；
- 4）调度线程，在子线程中进行耗时操作任务，在主线程中更新 UI ；
- 5）通过 subscribe()，根据请求成功或者失败来更新 UI 。
`这可能是最好的RxJava 2.x 入门教程（五） - 简书`
https://www.jianshu.com/p/81fac37430dd

- RxJava,Android 上用的比较多
- RxJava在服务端是否有使用场景和优势
```
1.Hystrix使用RxJava简洁的window API来构建metric应该算是一种不错的后端使用场景，说实话, RxJava虽然很酷, 但服务端使用RxJava的优势真心很少.
2.主要的原因还是大多数的Java服务端还是以同步逻辑为主, 迁移成本太高了.RxJava的响应式优势只有在异步逻辑占主导时才会体现出来. 异步和同步的夹杂使用, 还不如整体使用NodeJS的异步处理协调.其次, RxJava一大堆的数据处理API对习惯了同步逻辑的程序员来说, 学习成本也是相当高的.再加上后端的类库大多都是同步的API, 兼容RxJava的API的类库寥寥无几.所以基于RxJava的后端类库也是少之又少.
2.目前后端基于RxJava构建的最著名的类库是Hystrix, 它提供的API也是通过Command模式来作为同步的方式来调用.外部调用者无需关心内部的RxJava实现. 这样做应该也是为了降低使用者学习成本吧.

https://segmentfault.com/q/1010000004704554
```

- `Rxjava实际应用场景`
```
Scheduler线程切换——eg:后台线程取数据，主线程展示
CheckBox状态实时更新
输入框过滤——eg:登录注册等常用过滤
倒计时——eg:启动页或发送验证码
防抖动
https://www.jianshu.com/p/7b43a0883e62
```

---
# 差异
`这可能是最好的RxJava 2.x 入门教程(一) ` 与RxJava 1.x的差异
https://www.jianshu.com/p/a93c79e9f689

---
**参考**
`这可能是最好的RxJava 2.x 教程（完结版）`
https://www.jianshu.com/p/0cd258eecf60

