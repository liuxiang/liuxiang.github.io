title: Reactive programming(响应式编程)学习
date: 2018-4-19 00:00:00
categories: Reactive
tags: [Reactive]

---

[TOC]

---
# 体验概述
- 编程模式(反应式)的改变
- 利用反应式编程方便对非阻塞实现的封装(WebFlux)
- ...

---

# 介绍
反应式编程（Reactive Programming）这种新的编程范式越来越受到开发人员的欢迎。在 Java 社区中比较流行的是 RxJava 和 RxJava 2。以及一个新的反应式编程库 Reactor。

ReactiveX是Reactive Extensions的缩写，一般简写为Rx。Rx是一个编程模型，目标是提供一致的编程接口，帮助开发者更方便的处理异步数据流，Rx库支持.NET、JavaScript和C++，Rx近几年越来越流行了，现在已经支持几乎全部的流行编程语言了。社区网站是 ReactiveX 。

- `RxJava / RxJava2`  (https://github.com/ReactiveX/RxJava)
- `Reactor3` (https://github.com/reactor/reactor-core)
    - `Spring WebFlux`(https://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/web-reactive.html#webflux)

## reactivex.io/doc
> http://reactivex.io/documentation/observable.html   
http://reactivex.io/documentation/operators.html
http://reactivex.io/documentation/single.html 
http://reactivex.io/documentation/subject.html
http://reactivex.io/documentation/scheduler.html

---
**参考**
`Java Reactive 异步与并发编程`
https://blog.csdn.net/pmlpml/article/details/70470416

`响应式编程（Reactive Programming）介绍`
https://www.tuicool.com/articles/7biMbqE

`Java 9 揭秘（17. Reactive Streams）`  - 响应式流（Reactive Streams）
https://www.tuicool.com/articles/RNZ7jui
https://www.cnblogs.com/IcanFixIt/p/7245377.html

`浅析Java响应式编程(Reactive Programming) - 云+社区 - 腾讯云`
https://cloud.tencent.com/developer/article/1099762

`ReactiveX - 知乎`
https://www.zhihu.com/topic/20027327/top-answers?page=1

---
## 反应式编程介绍
反应式编程来源于数据流和变化的传播，意味着由底层的执行模型负责通过数据流来自动传播变化。`比如求值一个简单的表达式 c=a+b，当 a 或者 b 的值发生变化时，传统的编程范式需要对 a+b 进行重新计算来得到 c 的值`。如果`使用反应式编程，当 a 或者 b 的值发生变化时，c 的值会自动更新`。反应式编程最早由 .NET 平台上的 Reactive Extensions (Rx) 库来实现。后来迁移到 Java 平台之后就产生了著名的 RxJava 库，并产生了很多其他编程语言上的对应实现。在这些实现的基础上产生了后来的反应式流（Reactive Streams）规范。该规范定义了反应式流的相关接口，并将集成到 Java 9 中。
在传统的编程范式中，我们一般通过迭代器（Iterator）模式来遍历一个序列。这种遍历方式是由调用者来控制节奏的，采用的是拉的方式。每次由调用者通过 next()方法来获取序列中的下一个值。使用反应式流时采用的则是推的方式，即常见的发布者-订阅者模式。当发布者有新的数据产生时，这些数据会被推送到订阅者来进行处理。在反应式流上可以添加各种不同的操作来对数据进行处理，形成数据处理链。这个以声明式的方式添加的处理链只在订阅者进行订阅操作时才会真正执行。
反应式流中第一个重要概念是负压（backpressure）。在基本的消息推送模式中，当消息发布者产生数据的速度过快时，会使得消息订阅者的处理速度无法跟上产生的速度，从而给订阅者造成很大的压力。当压力过大时，有可能造成订阅者本身的奔溃，所产生的级联效应甚至可能造成整个系统的瘫痪。负压的作用在于提供一种从订阅者到生产者的反馈渠道。订阅者可以通过 request()方法来声明其一次所能处理的消息数量，而生产者就只会产生相应数量的消息，直到下一次 request()方法调用。这实际上变成了推拉结合的模式。

---
## Reactor 简介
前面提到的 RxJava 库是 JVM 上反应式编程的先驱，也是反应式流规范的基础。RxJava 2 在 RxJava 的基础上做了很多的更新。不过 RxJava 库也有其不足的地方。RxJava 产生于反应式流规范之前，虽然可以和反应式流的接口进行转换，但是由于底层实现的原因，使用起来并不是很直观。RxJava 2 在设计和实现时考虑到了与规范的整合，不过为了保持与 RxJava 的兼容性，很多地方在使用时也并不直观。`Reactor 则是完全基于反应式流规范设计和实现的库`，没有 RxJava 那样的历史包袱，在使用上更加的直观易懂。Reactor 也是 Spring 5 中反应式编程的基础。学习和掌握 Reactor 可以更好地理解 Spring 5 中的相关概念。

`使用 Reactor 进行反应式编程`
https://www.ibm.com/developerworks/cn/java/j-cn-with-reactor-response-encode/index.html?lnk=hmhm

`Reactor Reactive编程介绍(1)`
https://www.tuicool.com/articles/ba2aye6
https://luyiisme.github.io/2017/02/11/spring-reactor-programing/?utm_source=tuicool&utm_medium=referral


## 示例:Reactor
`（4）Reactor 3快速上手——响应式Spring的道法术器-刘康的博客-51CTO博客`
http://blog.51cto.com/liukang/2090191

---
# Rxjava
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
`给初学者的RxJava2.0教程(二) - 简书` https://www.jianshu.com/p/8818b98c44e2
`可能是最好的 Rx 初学者教程` https://zhuanlan.zhihu.com/p/25552305

### RxJava2 vs RxJava1
- RxJava2最大的改动就是对于backpressure的处理，为此将原来的Observable拆分成了新的Observable和Flowable，同时其他相关部分也同时进行了拆分。https://www.jianshu.com/p/850af4f09b61
- rxjava1 到 rxjava2的一些类名和类的方法名发生了变化。因此在使用new方式来设置对象的时候，通过要把类名和方法名字一起改了，如果使用的是lambda表达式，那么基本不用改代码。[+变更事项]   https://blog.csdn.net/weixin_39595561/article/details/78463173
- RxJava1 跟 RxJava2 不能共存.[+变更事项]  https://www.jianshu.com/p/6d644ca1678f
- Subject是RxJava1.x中就有的，继承自Observable，所以不支持背压，Processor是RxJava2.x中新加入的，所以支持背压 [+变更事项]  https://blog.csdn.net/jeasonlzy/article/details/74269443
- RxJava2与RxJava的比较 https://blog.csdn.net/jianesrq0724/article/details/54892758

---
# Reactor3 / Spring WebFlux 

- `Reactor3` (https://github.com/reactor/reactor-core) / Spring WebFlux
    - http://projectreactor.io/docs
    - http://projectreactor.io/docs/core/release/api/
    - http://projectreactor.io/docs/core/release/reference/
    - http://projectreactor.mydoc.io/ `Reactor2中文版`
    - http://htmlpreview.github.io/?https://github.com/get-set/reactor-core/blob/master-zh/src/docs/index.html `Reactor3中文版`

`响应式编程库Reactor 3 Reference Guide参考文档中文版（跟进最新版） - CSDN博客`
https://blog.csdn.net/get_set/article/details/79471861

---
## WebFlux 简介
WebFlux 模块的名称是 spring-webflux，名称中的 Flux 来源于 Reactor 中的类 Flux。该模块中包含了对反应式 HTTP、服务器推送事件和 WebSocket 的客户端和服务器端的支持。对于开发人员来说，比较重要的是服务器端的开发，这也是本文的重点。在服务器端，WebFlux 支持两种不同的编程模型：第一种是 Spring MVC 中使用的基于 Java 注解的方式；第二种是基于 Java 8 的 lambda 表达式的函数式编程模型。这两种编程模型只是在代码编写方式上存在不同。它们运行在同样的反应式底层架构之上，因此在运行时是相同的。WebFlux 需要底层提供运行时的支持，WebFlux 可以运行在支持 Servlet 3.1 非阻塞 IO API 的 Servlet 容器上，或是其他异步运行时环境，如 Netty 和 Undertow。

---
## Spring WebFlux动机
为什么要创建Spring WebFlux？
部分答案是需要一个无阻塞的Web栈来处理少量线程的并发性，并用较少的硬件资源进行扩展。Servlet 3.1确实为非阻塞I / O提供了一个API。但是，使用它会导致Servlet API的其余部分在同步（Filter，Servlet）或阻塞（getParameter， getPart）中生效。这是一个新的公共API作为跨越任何非阻塞运行时的基础的动机。这一点很重要，因为诸如Netty这样的服务器已经在异步，非阻塞空间中很好地建立起来了。
        答案的另一部分是函数式编程。就像在Java 5中添加注释创建机会 - 例如带注释的REST控制器或单元测试一样，在Java 8中添加lambda表达式为Java中的功能性API创造了机会。这是非阻塞应用程序和延续风格的API的福音-如由普及CompletableFuture和ReactiveX，其允许异步逻辑的声明性组合物。在编程模型层面，Java 8使Spring WebFlux能够在带注释的控制器的同时提供功能性Web端点。

- Spring Web MVC -> Spring Data
- Spring WebFlux -> Spring Data Reactive

- 同步阻塞的【spring-webmvc + servlet + Tomcat】
- 变成了响应式的异步非阻塞的【spring-webflux + Reactor + Netty】
> 在Java 7推出异步I/O库，以及Servlet3.1增加了对异步I/O的支持之后，Tomcat等Servlet容器也随后开始支持异步I/O，然后Spring WebMVC也增加了对Reactor库的支持，所以上边第4）步如果不是将spring-boot-starter-web替换为spring-boot-starter-WebFlux，而是增加reactor-core的依赖的话，仍然可以用注解的方式开发基于Tomcat的响应式应用。
`（5）Spring WebFlux快速上手——响应式Spring的道法术器-刘康的博客-51CTO博客`
http://blog.51cto.com/liukang/2090198

```
# 仅参考
SpringMVC是同步阻塞的IO模型，在处理一个比较耗时的任务时,线程一直在任务完成，期间线程是阻塞状态
Spring WebFlux的异步非阻塞模型,在处理一个比较耗时的任务时,线程可以做别的事情，期间线程是非阻塞状态.  
`深入浅出Spring Webflux系列（一）`
https://baijiahao.baidu.com/s?id=1590054970815500024&wfr=spider&for=pc
```

- Reactive意义: 当同步阻塞实现时,没有什么意义.但当流式处理时,客户端能逐步接收服务端的数据,表现为`同步非阻塞`实现.
```
/**
 * localhost:8080/flux/times
 *
 * @return
 */
@GetMapping(value = "/times", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> times() {
    return Flux.interval(Duration.ofSeconds(1)) 
            .map(l -> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));// 无限流[服务端推送]
}
```

---
**参考**
`spring Reactive RESTful Web Service`
https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-module
https://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/web-reactive.html#webflux
https://docs.spring.io/spring-framework/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/web-reactive.html

`SpringOne 2017：与Pivotal聊大会、Spring、Reactor、WebFlux及其他 - 推酷`
https://www.tuicool.com/articles/V3aE7fM

`Web on Reactive Stack`
https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-module

`Spring 5 之 Spring Webflux 开发 Reactive 应用 - 可译网`
https://coyee.com/article/12086-spring-5-reactive-web

`使用 Spring 5 的 WebFlux 开发反应式 Web 应用` `★★★`
https://www.ibm.com/developerworks/cn/java/spring5-webflux-reactive/index.html

`Spring 5：使用 Spring Webflux 开发 Reactive 应用 - 技术翻译 - 开源中国社区`
https://www.oschina.net/translate/spring-5-reactive-web-services

---
## spring mvc,spring webflux 差异图示
- diagram-boot-reactor.svg
![](http://spring.io/img/homepage/diagram-boot-reactor.svg)

- spring-mvc-and-webflux-venn.png
![](https://docs.spring.io/spring/docs/current/spring-framework-reference/images/spring-mvc-and-webflux-venn.png)
![](https://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/images/webflux-overview.png)

左侧是传统的基于Servlet的Spring Web MVC框架，右侧是5.0版本新引入的基于Reactive Streams的Spring WebFlux框架，从上到下依次是Router Functions，WebFlux，Reactive Streams三个新组件。
- Router Functions: 对标@Controller，@RequestMapping等标准的Spring MVC注解，提供一套函数式风格的API，用于创建Router，Handler和Filter。
- WebFlux: 核心组件，协调上下游各个组件提供响应式编程支持。
- Reactive Streams: 一种支持背压（Backpressure）的异步数据流处理标准，主流实现有RxJava和Reactor，Spring WebFlux默认集成的是Reactor。
在Web容器的选择上，Spring WebFlux既支持像Tomcat，Jetty这样的的传统容器（前提是支持Servlet 3.1 Non-Blocking IO API），又支持像Netty，Undertow那样的异步容器。不管是何种容器，Spring WebFlux都会将其输入输出流适配成Flux<DataBuffer>格式，以便进行统一处理。
值得一提的是，除了新的Router Functions接口，Spring WebFlux同时支持使用老的Spring MVC注解声明Reactive Controller。和传统的MVC Controller不同，Reactive Controller操作的是非阻塞的`ServerHttpRequest`和`ServerHttpResponse`，而不再是Spring MVC里的`HttpServletRequest`和`HttpServletResponse`。
```
 @GetMapping("/reactive/restaurants")
    public Flux<Restaurant> findAll() {
        return restaurantRepository.findAll();
    }
```
可以看到主要变化就是在 返回的类型上Flux<Restaurant>
Flux和Mono 是 Reactor 中的流数据类型，其中Flux会发送多次，Mono会发送0次或一次
`spring5.0 函数式web框架 webflux - CSDN博客`
https://blog.csdn.net/qq_34438958/article/details/78539234

---
## Flux,Mono 图示
- `Flux`的工作模式，可以看出Flux可以emit很多item
Flux 相当于一个 RxJava Observable，能够发出 0~N 个数据项，然后（可选地）completing 或 erroring。
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-4-28/66036038.jpg)

- `Mono`只能emit最多只能emit一个item
Mono ,是指最多只能触发(emit) (事件)一次。它对应于 RxJava 库的 Single 和 Maybe 类型。因此一个异步任务，如果只是想要在完成时给出完成信号，就可以使用 Mono<Void> 。
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-4-28/62777616.jpg)

`Spring 5 WebFlux - 简书` Flux,Mono 图示
https://www.jianshu.com/p/40a0ebe321be

## Flux 类的静态方法
第一种方式是通过 Flux 类中的静态方法。
- just()：可以指定序列中包含的全部元素。创建出来的 Flux 序列在发布这些元素之后会自动结束。
- fromArray()，fromIterable()和 fromStream()：可以从一个数组、Iterable 对象或 Stream 对象中创建 Flux 对象。
- empty()：创建一个不包含任何元素，只发布结束消息的序列。
- error(Throwable error)：创建一个只包含错误消息的序列。
- never()：创建一个不包含任何消息通知的序列。
- range(int start, int count)：创建包含从 start 起始的 count 个数量的 Integer 对象的序列。
- interval(Duration period)和 interval(Duration delay, Duration period)：创建一个包含了从 0 开始递增的 Long 对象的序列。其中包含的元素按照指定的间隔来发布。除了间隔时间之外，还可以指定起始元素发布之前的延迟时间。
- intervalMillis(long period)和 intervalMillis(long delay, long period)：与 interval()方法的作用相同，只不过该方法通过毫秒数来指定时间间隔和延迟时间。

`使用 Reactor 进行反应式编程`
https://www.ibm.com/developerworks/cn/java/j-cn-with-reactor-response-encode/index.html?lnk=hmhm

`聊聊reactive streams的Mono及Flux - 简书`
https://www.jianshu.com/p/c3585abbb9f4
https://segmentfault.com/a/1190000012823481

---
## backpressure
RS 规范和 Reactor 本身的主要焦点之一是背压(backpressure)。在生产者（生成）比消费者（消费）更快的 push 场景前提下，背压原理是，让消费者以信号反馈到生产者说：“喂！慢一点，我处理不过来了。” 这使得生产者有机会控制其速度，而不必丢弃数据（采取抽样）或致更糟的失败。

此时，你可能会想到 Mono：单一事件发出对消费者处理而言是没有压力的。Mono 工作和 (java8） CompletableFuture 工作原理之间仍然有一个关键的区别。后者只是推送：如果你有一个持有个 Future，这意味着处理异步结果的任务已经在执行。另一方面，背压的Flux或Mono支持：延迟(deferred)的(pull-push)交互：
deferred（延迟），因为在调用 subscribe() 之前没有任何事发生；
拉（pull），因为在订阅和(request steps)请求步骤时, Subscriber 会向上游事件源发送信号，并且拉下一个数据块；
推（push）,从生产者推到消费者哪里，限制请求元素的数量；

对于 Mono 来说，subscribe()类似按下按钮，说“我准备好接收我的数据”。对于 Flux，这个按钮是 request(n)，这是一种前者的泛化。
意识到 Mono 是 一个（代价高昂的任务（如在IO，时延等方面））Publisher，对认识背压的价值而言至关重要：如果你不订阅，您不需要支付该任务的成本。由于 Mono 和背压的 Flux 经常被编排在 reactive 代码链上，可能组合来自多个异步源的结果，所以这种按需订阅触发能力是避免阻塞的关键。
有背压帮助理解一个用例：异步地将 Flux 里的数据聚合到一个 Mono。运算符如 reduce 和 hasElement 能够消费 Flux 的每一项，从中聚合出某种形式的数据（分别是 reduce 函数结果和一个 boolean 值），并将这些数据暴露为一个 Mono。在这种情况下，上游得到的背压信号是 Long.MAX_VALUE,使得上游以完全推的方式工作。

背压的另一个有趣的方面是它如何限制（数据源）流在内存中的对象数量。作为一个 Publisher，数据源在产生数据项时可能是缓慢的，所以对于来自下游的请求，它可以很好地开始产生超过已经就绪的数据项目数目。在这种情况下，整个流自然地回到推送模式（新项目直接通知给消费者）。但是，当有一个生产峰值，生产速度加快时，会很好地回到拉模型。在这两种情况下，最多将 N 数据（the request()量）保存在内存中。

通过需要的数据数量 N 与每项数据消耗的 kb 大小 W：您可以推断出最多 W*N 内存消耗。事实上，Reactor 大部分时间利通过知道 N 进行优化：创建相应地限制的队列并且应用预取策略，可以自动请求 N的75％（接收处理3/4的量）。

最后，Reactor 运算符有时会改变背压信号，将其与和运算符表示的期望和语义相关联。此行为的一个典型的例子是buffer(10)：对于每一个（来自下游）请求 N，即运算符将从上游请求 10N ，它代表以足够数据填充缓冲器的数量应对消费。这被称为“活性背压”，开发者在微-批处理场景，可以自己控制以明确地告诉 Reactor 如何从一定输入量切换调整到一个不同的输出量。

`Reactor Reactive编程介绍(2)-背压故事 - Luyi's Blog`
https://luyiisme.github.io/2017/02/12/spring-reactor-programing2/

`聊聊reactive streams的backpressure - 推酷`
https://www.tuicool.com/articles/fQnEVbQ

---
## Publisher
感觉就是将Http Service 抽象为了 Function<Request , Response<Publisher<T>>>， `Publisher就是Reactive 中常说的Observable或Stream`，这里又叫`Publisher (Flux&Mono)，Publisher负责了异步操作`.
对其架构进行猜测：前方为传统的accept线程池，分发请求，运行route functions，组合publishers 返回结果；后方为reactive的线程池（为RP提供异步支持），对于简单的操作直接返回Response<T>，不用后方的reactive线程；而对与DAO操作等耗时操作（返回Response<Publisher<T>>），则被异步化了
所以最大的变化是引入了Reactive Programming，（并且Rx系的库API很Functional），前面route的DSL和SpringMVC的Mapping没多大变化（顶多就是个monoid append） ；至于Reactive Programming将DAO操作（包括RPC调用等）异步化，比自己去对Future做combination要高一级，不会出大问题（并且Publisher是个Monad，使用得当不会发生Callback Hell）。

`如何看待Spring 5引入函数式编程思想以及Reactor? - 知乎`
https://www.zhihu.com/question/52567283/answer/194147153

---
## 长连接
`使用 Spring 5 的 WebFlux 开发反应式 Web 应用`
https://www.ibm.com/developerworks/cn/java/spring5-webflux-reactive/index.html

### 服务器推送事件（Server-Sent Events，SSE）(允许服务器端不断地推送数据到客户端)
- 服务端
```
    @GetMapping("/randomNumbers")
    public Flux<ServerSentEvent<Integer>> randomNumbers() {
        return Flux.interval(Duration.ofSeconds(1))
                .map(seq -> ***);
    }
```

### WebSocket (支持客户端与服务器端的双向通讯)
示例-`Getting Started · Using WebSocket to build an interactive web application`
https://spring.io/guides/gs/messaging-stomp-websocket/

`Spring Search:websocket`
https://spring.io/search?q=WebSocket

---

## 示例:Spring Webflux
- 官方示例
http://spring.io/guides/gs/reactive-rest-service/

`（5）Spring WebFlux快速上手——响应式Spring的道法术器-刘康的博客-51CTO博客`
http://blog.51cto.com/liukang/2090198

`spring webflux返回application/stream+json - 简书`
https://www.jianshu.com/p/9ac1ecbbdbcd

`Spring Boot 2.0 WebFlux 上手系列课程：快速入门（一） - 简书`
https://www.jianshu.com/p/3ccfca09dcd6

---
# 其它
- 相关`Reactor / Proactor` 见: `Reactor Proactor 事件模式[reactor/proactor].md`

