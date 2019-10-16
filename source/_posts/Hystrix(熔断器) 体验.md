title: Hystrix(熔断器) 体验
date: 2018-3-30 00:00:00
categories: Hystrix
tags: [Hystrix]

---

[TOC]

---
# 请求流程
![](http://www.iocoder.cn/images/Hystrix/2018_10_31/01.jpeg)
`springcloud入门系列(6)－Hystrix详解 - 简书`
https://www.jianshu.com/p/efb049107572

## 大致表现
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/18-4-27/18411187.jpg)

`熔断Hystrix使用尝鲜 - 掘金`
https://juejin.im/post/5aab23c05188257bf550cdd6?utm_source=tuicool&utm_medium=referral

---

## RxJava(观察者模式)
![](https://upload-images.jianshu.io/upload_images/648342-d00fb3e56f972794.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

---
# 构件
## 隔离模式(ExecutionIsolationStrategy)
- 线程隔离-线程池: THREAD
- 线程隔离-信号量: SEMAPHORE
见: com.netflix.hystrix.HystrixCommandProperties.ExecutionIsolationStrategy

## 熔断机制-熔断器(Circuit Breaker) [作用周期:最近10s(可配)]
- `CLOSED > OPEN`
    - `周期` : 最近10s(可配)内，总失败请求(超时/隔离)数超过20次(可配) 。
        - HystrixCommandProperties.default_metricsRollingStatisticalWindow = 10000 (ms)
        - HystrixCommandProperties.circuitBreakerRequestVolumeThreshold = 20 
    - `错误` : 最近10s(可配)内,错误请求数超过20次(可配)后,此时一定错误占比50%(可配)将开启熔断器,持续5s
        - HystrixCommandProperties.circuitBreakerErrorThresholdPercentage = 50%
        - HystrixCommandProperties.circuitBreakerSleepWindowInMilliseconds = 5000 (ms)
- `OPEN > HALF_OPEN > CLOSE/OPEN `
    - `恢复` : 当前时间超过断路器`开启`5s(可配)时间,断路器变成`HALF_OPEN`状态,会尝试调用正常逻辑,根据执行是否成功决定`打开或关闭`熔断器
- 建议结合HystrixDashboard观察数据变化.
    - 代码逻辑见:com.netflix.hystrix.HystrixCircuitBreaker.HystrixCircuitBreakerImpl#subscribeToStream

- 关系配置: com.netflix.hystrix.HystrixCommandProperties#HystrixCommandProperties  (更多详见源码或下文)
```
/* defaults */
/* package */ static final Integer default_metricsRollingStatisticalWindow = 10000;// default =>统计窗口:10000 = 10秒(默认为10个桶，所以每个桶是1秒)
private static final Integer default_metricsRollingStatisticalWindowBuckets = 10;//默认=>统计窗口:10秒窗口中的10个桶，每个桶是1秒。
private static final Integer default_circuitBreakerRequestVolumeThreshold = 20;// default =>统计数据:在统计数据之前，必须在10秒内发出20个请求。
private static final Integer default_circuitBreakerSleepWindowInMilliseconds = 5000;// default => sleepWindow: 5000 = 5秒，我们在试过之后再试一次。
private static final Integer default_circuitBreakerErrorThresholdPercentage = 50;// default => erroroldpercentage = 50 =如果在10秒内超过50%的请求是失败或潜在的，那么我们将访问该电路。
```
见:`熔断器 Hystrix 源码解析 —— 断路器 HystrixCircuitBreaker`
https://www.tuicool.com/articles/jYrayuj

## 回退降级
Hystrix的降级回退方式
- Fail Fast 快速失败
- Fail Silent 无声失败
- Fallback: Static 返回默认值
- Fallback: Stubbed 自己组装一个值返回
- Fallback: Cache via Network 利用远程缓存
- Primary + Secondary with Fallback 主次方式回退（主要和次要）
见: https://github.com/Netflix/Hystrix/wiki/How-To-Use#common-patterns

---
- 总结
Hystrix为我们提供了一套线上系统容错的技术实践方法，我们通过在系统中引入Hystrix的jar包可以很方便的使用线程隔离、熔断、回退等技术。同时它还提供了监控页面配置，方便我们管理查看每个接口的调用情况。像spring cloud这种微服务构建模式中也引入了Hystrix，我们可以放心使用Hystrix的线程隔离技术，来防止雪崩这种可怕的致命性线上故障。

`王新栋 | Hystrix技术解析`
https://yq.aliyun.com/articles/183592

`Home · Netflix/Hystrix Wiki`
https://github.com/Netflix/Hystrix/wiki
https://github.com/Netflix/Hystrix/wiki/How-it-Works

---
# 如何使用 & 如何工作
- 依赖开始: https://github.com/Netflix/Hystrix/wiki/Getting-Started
- 如何使用: https://github.com/Netflix/Hystrix/wiki/How-To-Use
- 如何工作: https://github.com/Netflix/Hystrix/wiki/How-it-Works
- 配置设置: https://github.com/Netflix/Hystrix/wiki/Configuration

---
# 关键源码
## 初始化: com.netflix.hystrix.AbstractCommand#AbstractCommand
- 对象组成:
```
this.commandGroup = initGroupKey(group);
this.commandKey = initCommandKey(key, getClass());
this.properties = initCommandProperties(this.commandKey, propertiesStrategy, commandPropertiesDefaults); `配置初始化`
this.threadPoolKey = initThreadPoolKey(threadPoolKey, this.commandGroup, this.properties.executionIsolationThreadPoolKeyOverride().get());
this.metrics = initMetrics(metrics, this.commandGroup, this.threadPoolKey, this.commandKey, this.properties);
this.circuitBreaker = initCircuitBreaker(this.properties.circuitBreakerEnabled().get(), circuitBreaker, this.commandGroup, this.commandKey, this.properties, this.metrics); `熔断器实现`
this.threadPool = initThreadPool(threadPool, this.threadPoolKey, threadPoolPropertiesDefaults);
//Strategies from plugins
this.eventNotifier = HystrixPlugins.getInstance().getEventNotifier();
this.concurrencyStrategy = HystrixPlugins.getInstance().getConcurrencyStrategy();
HystrixMetricsPublisherFactory.createOrRetrievePublisherForCommand(this.commandKey, this.commandGroup, this.metrics, this.circuitBreaker, this.properties);
this.executionHook = initExecutionHook(executionHook);
this.requestCache = HystrixRequestCache.getInstance(this.commandKey, this.concurrencyStrategy);
this.currentRequestLog = initRequestLog(this.properties.requestLogEnabled().get(), this.concurrencyStrategy);
/* fallback semaphore override if applicable */
this.fallbackSemaphoreOverride = fallbackSemaphore;
/* execution semaphore override if applicable */
this.executionSemaphoreOverride = executionSemaphore;
```

## 断路器实现: com.netflix.hystrix.HystrixCircuitBreaker.HystrixCircuitBreakerImpl
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/18-4-27/33464218.jpg)
- 函数
    - subscribeToStream()
    - attemptExecution()
    - markSuccess()
    - markNonSuccess()
    - allowRequest()
    - isOpen()
见:`HystrixCircuitBreaker源码分析` https://cloud.tencent.com/developer/article/1049592

- 断路器状态控制: com.netflix.hystrix.HystrixCircuitBreaker.HystrixCircuitBreakerImpl#subscribeToStream
```
/*
 * This stream will recalculate the OPEN/CLOSED status on every onNext from the health stream
 */
return metrics.getHealthCountsStream()
        .observe()
        .subscribe(new Subscriber<HealthCounts>() {
            @Override
            public void onCompleted() {

            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onNext(HealthCounts hc) {
                // check if we are past the statisticalWindowVolumeThreshold
                if (hc.getTotalRequests() < properties.circuitBreakerRequestVolumeThreshold().get()) {
                    // we are not past the minimum volume threshold for the stat window,
                    // so no change to circuit status.
                    // if it was CLOSED, it stays CLOSED
                    // if it was half-open, we need to wait for a successful command execution
                    // if it was open, we need to wait for sleep window to elapse
                } else {
                    if (hc.getErrorPercentage() < properties.circuitBreakerErrorThresholdPercentage().get()) {
                        //we are not past the minimum error threshold for the stat window,
                        // so no change to circuit status.
                        // if it was CLOSED, it stays CLOSED
                        // if it was half-open, we need to wait for a successful command execution
                        // if it was open, we need to wait for sleep window to elapse
                    } else {
                        // our failure rate is too high, we need to set the state to OPEN
                        if (status.compareAndSet(Status.CLOSED, Status.OPEN)) {
                            circuitOpened.set(System.currentTimeMillis());
                        }
                    }
                }
            }
        });
```
`CircuitBreaker模式的Java实现 - code-craft - SegmentFault 思否`
https://segmentfault.com/a/1190000004980525

## Properties 初始化及默认值 : com.netflix.hystrix.HystrixCommandProperties#HystrixCommandProperties
- org.springframework.cloud.netflix.zuul.filters.route.support.AbstractRibbonCommand#getSetter
配置值(com.netflix.hystrix.HystrixCommandProperties.Setter)
```
/* defaults */
/* package */ static final Integer default_metricsRollingStatisticalWindow = 10000;// default =>统计窗口:10000 = 10秒(默认为10个桶，所以每个桶是1秒)
private static final Integer default_metricsRollingStatisticalWindowBuckets = 10;//默认=>统计窗口:10秒窗口中的10个桶，每个桶是1秒。
private static final Integer default_circuitBreakerRequestVolumeThreshold = 20;// default =>统计数据:在统计数据之前，必须在10秒内发出20个请求。
private static final Integer default_circuitBreakerSleepWindowInMilliseconds = 5000;// default => sleepWindow: 5000 = 5秒，我们在试过之后再试一次。
private static final Integer default_circuitBreakerErrorThresholdPercentage = 50;// default => erroroldpercentage = 50 =如果在10秒内超过50%的请求是失败或潜在的，那么我们将访问该电路。
private static final Boolean default_circuitBreakerForceOpen = false;// default => forceCircuitOpen = false(我们希望允许流量)
/* package */ static final Boolean default_circuitBreakerForceClosed = false;// default => ignoreerror = false。
private static final Integer default_executionTimeoutInMilliseconds = 1000; // default => executiontimeoutin毫秒:1000 = 1秒。
private static final Boolean default_executionTimeoutEnabled = true;
private static final ExecutionIsolationStrategy default_executionIsolationStrategy = ExecutionIsolationStrategy.THREAD;
private static final Boolean default_executionIsolationThreadInterruptOnTimeout = true;
private static final Boolean default_executionIsolationThreadInterruptOnFutureCancel = false;
private static final Boolean default_metricsRollingPercentileEnabled = true;
private static final Boolean default_requestCacheEnabled = true;
private static final Integer default_fallbackIsolationSemaphoreMaxConcurrentRequests = 10;
private static final Boolean default_fallbackEnabled = true;
private static final Integer default_executionIsolationSemaphoreMaxConcurrentRequests = 10;
private static final Boolean default_requestLogEnabled = true;
private static final Boolean default_circuitBreakerEnabled = true;
private static final Integer default_metricsRollingPercentileWindow = 60000; //默认为1分钟滚动百分比。
private static final Integer default_metricsRollingPercentileWindowBuckets = 6; //默认为6个桶(60秒为10秒)
private static final Integer default_metricsRollingPercentileBucketSize = 100; //默认值为100。
private static final Integer default_metricsHealthSnapshotIntervalInMilliseconds = 500; //默认为500ms，在允许健康快照(错误百分比等)之间的最大频率

@SuppressWarnings("unused") private final HystrixCommandKey key;
private final HystrixProperty<Integer> circuitBreakerRequestVolumeThreshold; //在使用统计数据进行公开/关闭决策之前，必须在统计窗口内发出的请求数量。
private final HystrixProperty<Integer> circuitBreakerSleepWindowInMilliseconds; //在允许重试之前，先断开电路后的毫秒数。
private final HystrixProperty<Boolean> circuitBreakerEnabled; //是否应该启用断路器。
private final HystrixProperty<Integer> circuitBreakerErrorThresholdPercentage; // %的“标记”，必须不能访问该电路。
private final HystrixProperty<Boolean> circuitBreakerForceOpen; //一个允许强制电路打开的属性(停止所有请求)
private final HystrixProperty<Boolean> circuitBreakerForceClosed; //一个允许忽略错误的属性，因此永远不会打开“打开”(ie)。允许所有流量通过)
private final HystrixProperty<ExecutionIsolationStrategy> executionIsolationStrategy; //是否应该在单独的线程中执行命令。
private final HystrixProperty<Integer> executionTimeoutInMilliseconds; //以毫秒为单位的超时值。
private final HystrixProperty<Boolean> executionTimeoutEnabled; //是否应该触发超时。
private final HystrixProperty<String> executionIsolationThreadPoolKeyOverride; //该命令应该运行的线程池(如果在单独的线程上运行)。
private final HystrixProperty<Integer> executionIsolationSemaphoreMaxConcurrentRequests; //执行信号的许可证数量。
private final HystrixProperty<Integer> fallbackIsolationSemaphoreMaxConcurrentRequests; //回退信号的许可证数量。
private final HystrixProperty<Boolean> fallbackEnabled; //是否应该尝试撤退。
private final HystrixProperty<Boolean> executionIsolationThreadInterruptOnTimeout; //一个潜在的将来/线程(当runinsecatethread == true)应该在超时后被中断。
private final HystrixProperty<Boolean> executionIsolationThreadInterruptOnFutureCancel; //是否取消潜在的将来/线程(当runinsecatethread == true)应该中断执行线程。
private final HystrixProperty<Integer> metricsRollingStatisticalWindowInMilliseconds; //毫秒将被跟踪。
private final HystrixProperty<Integer> metricsRollingStatisticalWindowBuckets; //统计窗口中的桶数。
private final HystrixProperty<Boolean> metricsRollingPercentileEnabled; //是否应该启用监视(SLA和Tracers)。
private final HystrixProperty<Integer> metricsRollingPercentileWindowInMilliseconds; //将在RollingPercentile中跟踪的毫秒数。
private final HystrixProperty<Integer> metricsRollingPercentileWindowBuckets; //桶数百分比窗口将被分成。
private final HystrixProperty<Integer> metricsRollingPercentileBucketSize; //在每个百分比窗口中存储多少个值。
private final HystrixProperty<Integer> metricsHealthSnapshotIntervalInMilliseconds; //在健康快照之间的时间。
private final HystrixProperty<Boolean> requestLogEnabled; //是否启用了命令请求日志记录。
private final HystrixProperty<Boolean> requestCacheEnabled; //是否启用了请求缓存。
```
## 运行时值存储与监听: com.netflix.config.DynamicProperty.CachedValue#value

## 失败回退逻辑
- 信号量超出失败: com.netflix.hystrix.AbstractCommand#handleSemaphoreRejectionViaFallback
- 超时失败(&重试): com.netflix.client.ClientException.ErrorType#NUMBEROF_RETRIES_NEXTSERVER_EXCEEDED

`Hystrix 源码解析 —— 请求执行（四）之失败回退逻辑 | 芋道源码 —— 纯源码解析博客`
http://www.iocoder.cn/Hystrix/command-execute-fourth-fallback/

---
# 配置
见: https://github.com/Netflix/Hystrix/wiki/Configuration
- 使用熔断器(Circuit Breaker)
```
1、`circuitBreaker.enabled` 是否启用熔断器，默认是TURE。
2、`circuitBreaker.forceOpen` 熔断器强制打开，始终保持打开状态。默认值FLASE。
3、`circuitBreaker.forceClosed` 熔断器强制关闭，始终保持关闭状态。默认值FLASE。
4、`circuitBreaker.errorThresholdPercentage` 设定错误百分比，默认值50%，例如一段时间（10s）内有100个请求，其中有55个超时或者异常返回了，那么这段时间内的错误百分比是55%，大于了默认值50%，这种情况下触发熔断器-打开。
5、`circuitBreaker.requestVolumeThreshold`默认值20.意思是至少有20个请求才进行errorThresholdPercentage错误百分比计算。比如一段时间（10s）内有19个请求全部失败了。错误百分比是100%，但熔断器不会打开，因为requestVolumeThreshold的值是20. 这个参数非常重要
6、`circuitBreaker.sleepWindowInMilliseconds` 半开试探休眠时间，默认值5000ms。当熔断器开启一段时间之后比如5000ms，会尝试放过去一部分流量进行试探，确定依赖服务是否恢复。
```
- execution
```
# 策略参数设置
execution.isolation.strategy=THREAD|SEMAPHORE

# 是否开启超时，默认是true，开启
execution.timeout.enabled=?

# 发生超时是是否中断线程，默认是true。
execution.isolation.thread.interruptOnTimeout

#取消时是否中断线程，默认是false。
execution.isolation.thread.interruptOnCancel
```
- execution超时
```
# 超时时间
execution.isolation.thread.timeoutInMilliseconds

# 建议通过CommandKey设置不同微服务的超时时间,对于zuul而言，CommandKey就是service id：hystrix.command.[CommandKey].execution.isolation.thread.timeoutInMilliseconds=?

# 用来设置thread和semaphore两种隔离策略的超时时间，默认值是1000。
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=?
```
- SEMAPHORE 信号量
```
# execution-semaphore
# 这个值并非TPS、QPS、RPS等都是相对值，指的是1秒时间窗口内的事务/查询/请求，semaphore.maxConcurrentRequests是一个绝对值，无时间窗口，相当于亚毫秒级的，指任意时间点允许的并发数。当请求达到或超过该设置值后，其其余就会被拒绝。默认值是100。
execution.isolation.semaphore.maxConcurrentRequests=100

# 设置semaphore(信号量)熔断`[无效]`
优先级（1~4）由高到低：[无效,需要加入集群信息]
[优先级1]zuul.eureka.api.semaphore.maxSemaphores
[优先级2]zuul.semaphore.max-semaphores
[优先级3]hystrix.command.api.execution.isolation.semaphore.maxConcurrentRequests
[优先级4]hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests
见：https://www.jianshu.com/p/d401452fe76e

# 设置semaphore(信号量)熔断`[有效]` (需指定集群如:ribbon serviceId:f**s***-pro1) 
hystrix.command.f**s***-pro1.execution.isolation.semaphore.maxConcurrentRequests = 300
zuul.eureka.f**s***-pro1.semaphore.maxSemaphores=300

# fallback-semaphore
hystrix.command.HystrixCommandKey.fallback.isolation.semaphore.maxConcurrentRequests
hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests
```
完整: https://github.com/Netflix/Hystrix/wiki/Configuration
官译: https://www.jianshu.com/p/39763a0bd9b8

---
## 动态配置生效
- HystrixManager
```
public void init(String serverName) {

    ConcurrentCompositeConfiguration config = ((ConcurrentCompositeConfiguration) ConfigurationManager.getConfigInstance());

    int semaphore_maxConcurrent = getValueByCluster(serverName,
            shutterProperties.observe.semaphore_maxConcurrent_cluster_default,// 500
            shutterProperties.observe.semaphore_maxConcurrent_cluster);

    int hystrix_timeoutInMilliseconds = getValueByCluster(serverName,
            shutterProperties.observe.hystrix_timeoutInMilliseconds_cluster_default,// (60000+1001)*2
            shutterProperties.observe.hystrix_timeoutInMilliseconds_cluster);

    // 挑战者设置
    if (shutterProperties.single.cluster_challenger_key.equals(serverName)) {
        config.setOverrideProperty("hystrix.command." + serverName + ".execution.timeout.enabled", false);// 关闭超时熔断
        config.setOverrideProperty("hystrix.command." + serverName + ".circuitBreaker.forceClosed", true);// 强制关闭熔断器(即不熔断)
    }

    String strategyKey = "hystrix.command." + serverName + ".execution.isolation.strategy";// THREAD/SEMAPHORE
    String timeoutInMillisecondsKey = "hystrix.command." + serverName + ".execution.isolation.thread.timeoutInMilliseconds";// 熔断器超时时间
    String semaphore_maxConcurrentKey = "hystrix.command." + serverName + ".execution.isolation.semaphore.maxConcurrentRequests";// 最大执行并发数
    String semaphore_fallback_maxConcurrentKey = "hystrix.command." + serverName + ".fallback.isolation.semaphore.maxConcurrentRequests";// 最大失败并发数

    config.setOverrideProperty(strategyKey, HystrixCommandProperties.ExecutionIsolationStrategy.SEMAPHORE);// 方式: SEMAPHORE
    config.setOverrideProperty(semaphore_maxConcurrentKey, semaphore_maxConcurrent);// 最大并发数(信号量):500
    config.setOverrideProperty(semaphore_fallback_maxConcurrentKey, semaphore_maxConcurrent);// 最大并发数(信号量):500
    config.setOverrideProperty(timeoutInMillisecondsKey, hystrix_timeoutInMilliseconds);// 61s * 2

    /** 初始化 LoadBalancer (可选:在不指定初始化情况下,在该集群流量进来时会自动初始化) **/
    DynamicServerListLoadBalancer dynamicServerListLoadBalancer = (DynamicServerListLoadBalancer) springClientFactory.getLoadBalancer(serverName);// 运行实例
    dynamicServerListLoadBalancer.initWithNiwsConfig(dynamicServerListLoadBalancer.getClientConfig());
}
```

---
# 其它相关
## Fallback
- `获取Fallback`: org.springframework.cloud.netflix.zuul.filters.route.support.AbstractRibbonCommandFactory#getFallbackProvider
```
protected ZuulFallbackProvider getFallbackProvider(String route) {
    ZuulFallbackProvider provider = fallbackProviderCache.get(route);
    if(provider == null) {
        provider = defaultFallbackProvider;
    }
    return provider;
}
```

- `执行Fallback`: org.springframework.cloud.netflix.zuul.filters.route.support.AbstractRibbonCommand#getFallback
```
public abstract class AbstractRibbonCommand<LBC extends AbstractLoadBalancerAwareClient<RQ, RS>, RQ extends ClientRequest, RS extends HttpResponse>
        extends HystrixCommand<ClientHttpResponse> implements RibbonCommand {

    @Override
    protected ClientHttpResponse getFallback() {
        if(zuulFallbackProvider != null) {
            return zuulFallbackProvider.fallbackResponse();
        }
        return super.getFallback();
    }

}
```

- `自定义`: DefaultFallbackProvider
```
import org.springframework.cloud.netflix.zuul.filters.route.ZuulFallbackProvider;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.client.ClientHttpResponse;
import org.springframework.stereotype.Component;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.Charset;

@Component
public class DefaultFallbackProvider implements ZuulFallbackProvider {
  
    @Override  
    public String getRoute() {  
        return null; // 即,默认
        //见:org.springframework.cloud.netflix.zuul.filters.route.support.AbstractRibbonCommandFactory.AbstractRibbonCommandFactory
    }  
  
    @Override  
    public ClientHttpResponse fallbackResponse() {
        return new ClientHttpResponse() {  
            @Override  
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }  
  
            @Override  
            public int getRawStatusCode() throws IOException {  
                return this.getStatusCode().value();  
            }  
  
            @Override  
            public String getStatusText() throws IOException {  
                return this.getStatusCode().getReasonPhrase();  
            }  
  
            @Override  
            public void close() {  
  
            }  
  
            @Override  
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("Service不可用".getBytes());
            }  
  
            @Override  
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();  
                MediaType mt = new MediaType("application", "json", Charset.forName("UTF-8"));
                headers.setContentType(mt);  
                return headers;  
            }  
        };  
    }  
}  
```
https://blog.csdn.net/tianyaleixiaowu/article/details/78064127
https://blog.csdn.net/qq_24504315/article/details/79121938

---
## Metrics-and-Monitoring
见: https://github.com/Netflix/Hystrix/wiki/Metrics-and-Monitoring

---
## HystrixPlugins
见:https://github.com/Netflix/Hystrix/wiki/Plugins

---
## Hystrix Dashboard
详见: https://github.com/Netflix/Hystrix/wiki/Operations

- 开启注解: @EnableHystrixDashboard 
    - 集群监听: @EnableTurbine // 启用收集断路器集群服务功能
- 访问: http://localhost:8080/hystrix/monitor?stream=http://localhost:8080/hystrix.stream
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/18-4-28/3796358.jpg)
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/18-4-28/10125367.jpg)

---
**参考**
`史上最简单的SpringCloud教程 | 第四篇:断路器（Hystrix） - CSDN博客` - Feign中使用断路器
https://blog.csdn.net/forezp/article/details/69934399

`springcloud入门系列(6)－Hystrix详解 - 简书`
https://www.jianshu.com/p/efb049107572

`王新栋 | Hystrix技术解析(图片版) - 简书`
https://www.jianshu.com/p/b6e8d91b2a96

**综合**
`zuul 参数调优 - 简书`
https://www.jianshu.com/p/d401452fe76e

`Hystrix入门 - 简书`
https://www.jianshu.com/p/828965e6e496

`防雪崩利器：熔断器 Hystrix 的原理与使用`
https://segmentfault.com/a/1190000005988895