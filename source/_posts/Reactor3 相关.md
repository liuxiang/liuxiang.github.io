title: Reactor3 相关
date: 2018-5-16 00:00:02
categories: Reactor3
tags: [Reactor3]

---

[TOC]

---
# 介绍
Reactor 是一个基于 JVM 之上的异步应用基础库。为 Java 、Groovy 和其他 JVM 语言提供了构建基于事件和数据驱动应用的抽象库。Reactor 性能相当高，在最新的硬件平台上，使用无堵塞分发器每秒钟可处理 1500 万事件。
Reactor 是一个基础库，可用它构建时效性流式数据应用，或者有低延迟和容错性要求的微/纳/皮级服务。

github: https://github.com/reactor/reactor-core
- http://projectreactor.io/docs
- http://projectreactor.io/docs/core/release/api/
- http://projectreactor.io/docs/core/release/reference/
- http://projectreactor.mydoc.io/ `Reactor2中文版`
- http://htmlpreview.github.io/?https://github.com/get-set/reactor-core/blob/master-zh/src/docs/index.html `Reactor3中文版`

`响应式编程库Reactor 3 Reference Guide参考文档中文版（跟进最新版） - CSDN博客`
https://blog.csdn.net/get_set/article/details/79471861

## 建议 (见:Reactor2中文版)
Reactor 旨在帮助大多数用例真正非阻塞地运行。我们提供的 API 比 JDK 的 java.util.concurrent 库低级原语更高效。Reactor 提供了下列功能的替代函数 (并建议不使用 JDK 原生语句)：
```
阻塞等待：如 Future.get()
不安全的数据访问：如 ReentrantLock.lock()
异常冒泡：如 try…catch…finally
同步阻塞：如 synchronized{ }
Wrapper分配(GC 压力)：如 new Wrapper<T>(event)
```
## 常规`使用线程池`其中的问题
```
private ExecutorService threadPool = Executors.newFixedThreadPool(8);

final List<T> batches = new ArrayList<T>();

Callable<T> t = new Callable<T>() { // *1

        public T run() {
                synchronized(batches) { // *2
                        T result = callDatabase(msg); // *3
                        batches.add(result);
                        return result;
                }
        }
};

Future<T> f = threadPool.submit(t); // *4
T result = f.get() // *5
```
- 带来的问题
```
1.Callable 分配 -- 可能导致 GC 压力。
2.同步过程强制每个线程执行停-检查操作。
3.消息的消费可能比生产慢。
4.使用线程池(ThreadPool)将任务传递给目标线程 -- 通过 FutureTask 方式肯定会产生 GC 压力。
5.阻塞直至 callDatabase() 回调。
```
- Reactor改善
```
Reactor 提供的框架可以帮助减轻应用中由延迟产生的副作用，只需要增加一点点开销：
- 使用了一些聪明的结构，通过启动`预分配`策略解决`运行时分配`问题；
- 通过确定信息传递`主结构的边界`，避免任务的`无限堆叠`；
- 采用主流的`响应与事件驱动`构架模式，提供包含反馈在内的`非阻塞端对端流`；
- 引入新的 `Reactive Streams标准`，`拒绝超过当前容量请求`，从而保证限制结构的有效性；
- 在IPC上也使用了类似理念，提供对`流控制友好的非阻塞 IO 驱动`；
- 开放了帮助开发者们以`零副作用`方式组织他们代码的函数接口，借助这些函数来处理`容错性和线程安全`。
```

---
## 场景
- reactor-数据流
- 微批处理
- 微服务(非阻塞服务,响应式背压)
- reactor-总线
- reactor-网络(异步 TCP、UDP 及 HTTP)
参考: http://projectreactor.mydoc.io/?t=44480

---
## 回调地狱（Callback Hell）的例子
详见: https://htmlpreview.github.io/?https://github.com/get-set/reactor-core/blob/master-zh/src/docs/index.html#_%E5%BC%82%E6%AD%A5%E5%8F%AF%E4%BB%A5%E8%A7%A3%E5%86%B3%E9%97%AE%E9%A2%98%E5%90%97
```
userService.getFavorites(userId, new Callback<List<String>>() { 
  public void onSuccess(List<String> list) { 
    if (list.isEmpty()) { 
      suggestionService.getSuggestions(new Callback<List<Favorite>>() {
        public void onSuccess(List<Favorite> list) { 
          UiUtils.submitOnUiThread(() -> { 
            list.stream()
                .limit(5)
                .forEach(uiList::show); 
            });
        }

        public void onError(Throwable error) { 
          UiUtils.errorPopup(error);
        }
      });
    } else {
      list.stream() 
          .limit(5)
          .forEach(favId -> favoriteService.getDetails(favId, 
            new Callback<Favorite>() {
              public void onSuccess(Favorite details) {
                UiUtils.submitOnUiThread(() -> uiList.show(details));
              }

              public void onError(Throwable error) {
                UiUtils.errorPopup(error);
              }
            }
          ));
    }
  }

  public void onError(Throwable error) {
    UiUtils.errorPopup(error);
  }
});
```
- 使用 Reactor 实现以上回调方式同样功能的例子
```
userService.getFavorites(userId) 
           .flatMap(favoriteService::getDetails) 
           .switchIfEmpty(suggestionService.getSuggestions()) 
           .take(5) 
           .publishOn(UiUtils.uiThreadScheduler()) 
           .subscribe(uiList::show, UiUtils::errorPopup); 
```

---
## 从命令式编程到响应式编程
类似 Reactor 这样的响应式库的目标就是要弥补上述“经典”的 JVM 异步方式所带来的不足， 此外还会关注一下几个方面：
- 可编排性（Composability） 以及 可读性（Readability）
- 使用丰富的 操作符 来处理形如 流 的数据
- `在 订阅（subscribe） 之前什么都不会发生`
- 背压（backpressure） 具体来说即 消费者能够反向告知生产者生产内容的速度的能力
- 高层次 （同时也是有高价值的）的抽象，从而达到 并发无关 的效果
-  热（Hot） vs 冷（Cold）
    - [冷]对于每一个 Subscriber，都会收到从头开始所有的数据
    - [热]对于一个 Subscriber，只能获取从它开始 订阅 之后 发出的数据

---
# 特性
## Flux,包含 0-N 个元素的异步序列
![](https://raw.githubusercontent.com/reactor/reactor-core/v3.0.7.RELEASE/src/docs/marble/flux.png)
```
Flux.range(1, 3).doOnNext(System.out::print).subscribe(System.out::print);
```

## Mono, 异步的 0-1 结果
![](https://raw.githubusercontent.com/reactor/reactor-core/v3.0.7.RELEASE/src/docs/marble/mono.png)
```
Mono.just("x").doOnNext(System.out::print).subscribe(System.out::print);
```

## Schedulers
```
Flux.interval(Duration.ofMillis(300), Schedulers.newSingle("test"))
        .doOnNext(System.out::print)
        .take(10)// 随仅取10个,但并不会中断流
        .subscribe((l)->{
            System.out.print(l);
            // 能否在订阅的中断流?
        });
```
## ParallelFlux
```
Mono.fromCallable( () -> System.currentTimeMillis() )
    .repeat()
    .parallel(8) //parallelism
    .runOn(Schedulers.parallel())
    .doOnNext( d -> System.out.println("I'm on thread "+Thread.currentThread()) )
    .subscribe()
```

## Backpressure
```
Flux.range(1, 10)
        .doOnNext(System.out::print)
        .map((n) -> String.valueOf(n))
        .subscribe(new BaseSubscriber<String>() {
            @Override
            protected void hookOnSubscribe(Subscription subscription) {
                request(1);// 通过 subscription 向上游传递 背压请求。这里我们在开始这个流的时候请求1个元素值。
            }

            @Override
            protected void hookOnNext(String value) {
                request(1);// 随着接收到新的值，我们继续以每次请求一个元素的节奏从源头请求值。
                System.out.println(value);
            }
        });
```

## 调试
- 典型的 Reactor Stack Trace
```
java.lang.IndexOutOfBoundsException: Source emitted more than one item
    at reactor.core.publisher.MonoSingle$SingleSubscriber.onNext(MonoSingle.java:120)
    at reactor.core.publisher.FluxFlatMap$FlatMapMain.emitScalar(FluxFlatMap.java:380)
    at reactor.core.publisher.FluxFlatMap$FlatMapMain.onNext(FluxFlatMap.java:349)
    at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:119)
    at reactor.core.publisher.FluxRange$RangeSubscription.slowPath(FluxRange.java:144)
    at reactor.core.publisher.FluxRange$RangeSubscription.request(FluxRange.java:99)
    at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.request(FluxMapFuseable.java:172)
    at reactor.core.publisher.FluxFlatMap$FlatMapMain.onSubscribe(FluxFlatMap.java:316)
    at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onSubscribe(FluxMapFuseable.java:94)
    at reactor.core.publisher.FluxRange.subscribe(FluxRange.java:68)
    at reactor.core.publisher.FluxMapFuseable.subscribe(FluxMapFuseable.java:67)
    at reactor.core.publisher.FluxFlatMap.subscribe(FluxFlatMap.java:98)
    at reactor.core.publisher.MonoSingle.subscribe(MonoSingle.java:58)
    at reactor.core.publisher.Mono.subscribeWith(Mono.java:2668)
    at reactor.core.publisher.Mono.subscribe(Mono.java:2629)
    at reactor.core.publisher.Mono.subscribe(Mono.java:2604)
    at reactor.core.publisher.Mono.subscribe(Mono.java:2582)
    at reactor.guide.GuideTests.debuggingCommonStacktrace(GuideTests.java:722)
```
- Reactor 内置了一种面向调试的能力—— 操作期测量（assembly-time instrumentation）
```
在应用启动的时候 （或至少在有问题的 Flux 或 Mono 实例化之前） 加入自定义的 Hook.onOperator 钩子（hook），如下：
Hooks.onOperatorDebug();
```
- 阅读调试模式的 Stack Trace
```
java.lang.IndexOutOfBoundsException: Source emitted more than one item
    at reactor.core.publisher.MonoSingle$SingleSubscriber.onNext(MonoSingle.java:120)
    at reactor.core.publisher.FluxOnAssembly$OnAssemblySubscriber.onNext(FluxOnAssembly.java:314) 
    ...
    at reactor.core.publisher.Mono.subscribeWith(Mono.java:2668)
    at reactor.core.publisher.Mono.subscribe(Mono.java:2629)
    at reactor.core.publisher.Mono.subscribe(Mono.java:2604)
    at reactor.core.publisher.Mono.subscribe(Mono.java:2582)
    at reactor.guide.GuideTests.debuggingActivated(GuideTests.java:727)
    Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Assembly trace from producer [reactor.core.publisher.MonoSingle] : 
    reactor.core.publisher.Flux.single(Flux.java:5335)
    reactor.guide.GuideTests.scatterAndGather(GuideTests.java:689)
    reactor.guide.GuideTests.populateDebug(GuideTests.java:702)
    org.junit.rules.TestWatcher$1.evaluate(TestWatcher.java:55)
    org.junit.rules.RunRules.evaluate(RunRules.java:20)
Error has been observed by the following operator(s): 
    |_ Flux.single(TestWatcher.java:55) 
```
- 用 checkpoint() 方式替代
- 记录流的日志 log()

## 高级特性与概念
见:https://htmlpreview.github.io/?https://github.com/get-set/reactor-core/blob/master-zh/src/docs/index.html#advanced

---
**参考**
`github` https://github.com/reactor/reactor-core
`Reactor3中文版` https://htmlpreview.github.io/?https://github.com/get-set/reactor-core/blob/master-zh/src/docs/index.html#core-features

`响应式Spring的道法术器（Spring WebFlux 教程）`
https://blog.csdn.net/get_set/article/details/79466657
```
Spring WebFlux 2小时快速入门, Spring 5 之使用Spring WebFlux开发响应式应用。
lambda与函数式（15min）
Reactor 3 响应式编程库（60min）
Spring Webflux和Spring Data Reactive开发响应式应用（45min）
```