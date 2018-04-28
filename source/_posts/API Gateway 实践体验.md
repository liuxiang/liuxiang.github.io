title: API Gateway 实践体验
date: 2018-3-15 00:00:00
categories: API Gateway
tags: [API Gateway]

---

[TOC]

---
# 一.API Gateway 场景及作用
初衷: 企业微服务,统一`流量管理,前后置任务,动态路由,安全防护`等API相关的通用能力.

- 前置（Pre）
    - 鉴权、安全、增加请求参数,恶意请求拦截
    - 隔离机制 (Hystrix | 限流(ip,url,user)-线程池,信号量)
- 路由（Route）
    - OriginServer(源服务访问)、请求转发、流量拷贝
    - 重试机制(幂等考虑:只重试网络错误)
    - 灰度发布
- 后置（Post）
    - 统计返回值和调用时间、记录日志、增加跨域头等
- 错误（Error）
    - 过程中断

## 业务网关
当业务系统所在分布式,集群组合环境中,如需对流量进行统一管理.那么业务API Gateway就出现了.
> 动态路由、业务数据过滤、统一鉴权、流量复制、环境差异(dev,test,production). 等业务视角的API Gateway能力

![](https://images2015.cnblogs.com/blog/1099841/201706/1099841-20170630111344414-1260445909.png)

## 常见选型
对于 API Gateway，常见的选型有基于 `Openresty 的 Kong`、基于 `Go 的 Tyk `和基于 `Java 的 Zuul`.
基本是技术栈的差异.

- OpenResty Kong
> OpenResty® 是一个基于 Nginx 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关
详见: http://openresty.org/cn/

- Zuul
> Zuul通过使用其他Netflix OSS组件给予我们很多洞察力，灵活性和灵活性：
`Hystrix` 被用来包装呼叫到我们的来源，这使我们能够在发生问题时减少流量并划分优先级
`Ribbon` 用于处理来自Zuul的所有出站请求，它提供了有关网络性能和错误的详细信息，并处理负载均衡的软件负载平衡
`Turbine` 实时聚合精细度量指标，以便我们可以快速观察并对问题做出反应
`Archaius` 处理配置并提供动态更改属性的功能
详见: https://github.com/Netflix/zuul/wiki/How-We-Use-Zuul-At-Netflix

---
# 二.Zuul 概述
- 技术本质: web servlet应用.  `入口: ZuulController extends ServletWrappingController`
- 框架能力: 实现了基础的流操作(接管,解析,转发,异常控制), 还设计以filter为切面,供开发者按需扩展的机制.

## 过滤器,机制
![](http://blog.didispace.com/content/images/posts/spring-cloud-zuul-exception-3-4.png)

- 内置过滤器:

| 类型 | 顺序 | 过滤器 | 功能 
| --- | --- | --- | --- 
| pre | -3 | ServletDetectionFilter | 标记处理Servlet的类型 
| pre | -2 | Servlet30WrapperFilter | 包装HttpServletRequest请求 
| pre | -1 | FormBodyWrapperFilter | 包装请求体 
| pre | 1 | DebugFilter | 标记调试标志 
| pre | 5 | PreDecorationFilter | 处理请求上下文供后续使用 
| route | 10 | RibbonRoutingFilter | serviceId请求转发 
| route | 100 | SimpleHostRoutingFilter | url请求转发 
| route | 500 | SendForwardFilter | forward请求转发 
| post | 0 | SendErrorFilter | 处理有错误的请求响应 
| post | 500 | SendForwardFilter | 处理forward
| post | 1000 | SendResponseFilter | 处理正常的请求响应 

详见:org.springframework.cloud.netflix.zuul.filters.support.FilterConstants#DEBUG_FILTER_ORDER

## 扩展实现方向
- `验证与安全保障`: 识别面向各类资源的验证要求并拒绝那些与要求不符的请求。
- `审查与监控`: 在边缘位置追踪有意义数据及统计结果，从而为我们带来准确的生产状态结论。
- `动态路由`: 以动态方式根据需要将请求路由至不同后端集群处。
- `压力测试`: 逐渐增加指向集群的负载流量，从而计算性能水平。
- `负载分配`: 为每一种负载类型分配对应容量，并弃用超出限定值的请求。
- `静态响应处理`: 在边缘位置直接建立部分响应，从而避免其流入内部集群。
- `多区域弹性`跨越AWS区域进行请求路由，旨在实现ELB使用多样化并保证边缘位置与使用者尽可能接近。

`单个实现均可以独立展开总结`

---
# 三.Zuul 应用
## 系统设计
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-3-19/45541994.jpg)

## 功能
- 集群及节点管理:对接triton，定时刷新-可配
- 健康检查:故障节点管理(失效,恢复机制)
- 负载均衡:权重（Weighted Round-Robin）
- 集群路由:合作方与集群关系 见:`SELECT business_code, partner_code,cluster FROM forseti.partner_cluster WHERE status=1;`
- 冠军挑战者: 见:`forseti.policy_set where challenged_uuid IS NOT NULL`
- 缓存技术方案:启动时加载,可配定时更新. [ChallengerCache,ClusterNodeCache,PartnerConfigCache]
- 流量拷贝(http Copy):见shutter配置`forseti-gateway-observe.properties`-`httpcopy.register:***`
- 内部请求头处理: 非内网访问清理测试标记
- 超时控制
- 环境控制:stg环境不需要httpCopy,且无集群路由(固定集群),但有挑战者集群
- 特殊合作方处理:大麦504转200
- 日志管理:整体控制日志量,access日志独立
- 监控管理:gateway,filter,业务,异常等.metrics采样,供grafana制图. 

---
## 基本实现
### 使用zuul filter
```
PreHeaderFilter
PrePartnerClusterFilter

RouteChallengerDispatchFilter
RouteHttpCopyFilter
RoutePartnerDispatchFilter

ErrorFilter

PostAccessLogFilter
PostHeaderFilter
```
- PreHeaderFilter
```
/**
 * 内部请求头处理(测试标记、超时)
 */
@SuppressWarnings("SpringJavaInjectionPointsAutowiringInspection")
@Service
public class PreHeaderFilter extends ZuulFilter {

    public final static Logger logger = LoggerFactory.getLogger(PreHeaderFilter.class);


    @Autowired
    ShutterObserve shutterObserve;

    @Autowired
    SimpleDiscoveryClient simpleDiscoveryClient;

    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE; // 可以在请求被路由之前调用
    }

    @Override
    public int filterOrder() {
        return FilterConstants.DEBUG_FILTER_ORDER+1;
    }

    @Override
    public boolean shouldFilter() {
        return true;// 是否执行该过滤器，此处为true，说明需要过滤
    }

    @Override
    public Object run() {

        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();

        // 当前会话的超时设置 TODO 依据合作方设置有困难

        // 仅办公网&内网支持.客户不支持 "service-type", "x-perf-test", "test-flag",对非公网测试标记访问,进行处理:剔除标记
        String clientHost = request.getRemoteHost();
        if (Arrays.asList(shutterObserve.getAddress_internal().split(",")).contains(clientHost)
                || IpUtil.isInRange(clientHost, "10.0.0.0/8")// 10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
                || IpUtil.isInRange(clientHost, "172.16.0.0/12")
                || IpUtil.isInRange(clientHost, "192.168.0.0/17")) {
            return null;
        }

        // 非内网访问,清理测试标记
        ctx.addZuulRequestHeader("service-type", null);
        ctx.addZuulRequestHeader("x-perf-test", null);
        ctx.addZuulRequestHeader("test-flag", null);

        return null;
    }

}
```

### 对接triton,用作服务发现 (替代方案:Netflix/eureka)
```
http://10.57.18.211/api/v1/apps/ips?app_name=forseti-api&env=production
```

eureka应用参考见:`微服务：Eureka+Zuul+Ribbon+Feign+Hystrix构建微服务架构`
http://blog.csdn.net/qq_18675693/article/details/53282031
wiki: https://github.com/Netflix/eureka/wiki/Configuring-Eureka

### 引入Ribbon,实现集群负载管理
```
# Max number of retries on the same server (excluding the first try)
sample-client.ribbon.MaxAutoRetries=1

# Max number of next servers to retry (excluding the first server)
sample-client.ribbon.MaxAutoRetriesNextServer=1

# Whether all operations can be retried for this client
sample-client.ribbon.OkToRetryOnAllOperations=true

# Interval to refresh the server list from the source
sample-client.ribbon.ServerListRefreshInterval=2000

# Connect timeout used by Apache HttpClient
sample-client.ribbon.ConnectTimeout=3000

# Read timeout used by Apache HttpClient
sample-client.ribbon.ReadTimeout=3000

# Initial list of servers, can be changed via Archaius dynamic property at runtime
sample-client.ribbon.listOfServers=www.microsoft.com:80,www.yahoo.com:80,www.google.com:80

详见:https://github.com/Netflix/ribbon/wiki/Getting-Started
```
- 示例:
```
# Routes定义`origin`
zuul.routes.origin.stripPrefix=false
zuul.routes.origin.path=/greeting/**
zuul.routes.origin.serviceId=originService

# Ribbon Service定义
originService.ribbon.NFLoadBalancerClassName=com.netflix.loadbalancer.DynamicServerListLoadBalancer
## 设置(默认:AvailabilityFilteringRule)
originService.ribbon.NFLoadBalancerRuleClassName=cn.fraudmetrix.forseti.gateway.rule.CustomWeightedResponseTimeRule

## Initial list of servers, can be changed via Archaius dynamic property at runtime
originService.ribbon.listOfServers=localhost:8090,localhost:8091

## 连接超时(default:2000) Connect timeout used by Apache HttpClient  
originService.ribbon.ConnectTimeout=1000

## 响应超时(default:5000) Read timeout used by Apache HttpClient
originService.ribbon.ReadTimeout=60000

详见: https://github.com/Netflix/ribbon/wiki/Working-with-load-balancers
```

### 异常响应控制 (org.springframework.boot.autoconfigure.web 提供)
```
@Component
public class CustomErrorAttribute extends DefaultErrorAttributes {
    @Override
    public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
        Map<String, Object> result = super.getErrorAttributes(requestAttributes, includeStackTrace);
        result.put("status", 500);
        result.put("error", "error");
        result.put("exception", "exception");
        result.put("message", "message");
        return result;
    }
}
```

### 自定义权重轮训
#### 基于权重随机
```
int n = 0, resultIndex = 0;
double random = new Random().nextDouble() * prizeArea.get(prizeArea.size()-1);
for (Integer inte : prizeArea) {
    if (inte >= random) {
        resultIndex = n;
        System.out.println("命中(" + random + "):" + inte);
        break;
    }
    n++;
}
```
#### 基于加权轮询
```
/**
 * 算法流程：
 * 假设有一组服务器 S = {S0, S1, …, Sn-1}
 * 有相应的权重，变量currentIndex表示上次选择的服务器
 * 权值currentWeight初始化为0，currentIndex初始化为-1 ，当第一次的时候返回 权值取最大的那个服务器，
 * 通过权重的不断递减 寻找 适合的服务器返回
 */
public Server getServer() {
    while (true) {
        currentIndex = (currentIndex + 1) % serverCount;// 每次从上一次命中节点,的下个节点开始权值比较
        if (currentIndex == 0) {// 第0台,第size台机器,取余才会得0 (让外部所有节点权值都进行权值比较,至找到合适的权值节点)

            // 当前权值减最大公约数(gcd)相比每次减(1,2,最大公差,n)的优点:`维持整数比例关系`的`最小值表现`.公约数可保持权重平衡,最大公约数可最快的轮换节点.
            currentWeight = currentWeight - gcdWeight;// max - gcd (权值衰减直到近一个合适的权重,才会换节点)

            if (currentWeight <= 0) {// 首次:0 - gcd = 负值 / 权重衰减至最大公约数时-gcd = 0值,此时需要重置最大权值
                currentWeight = maxWeight;// 为负时,取重置为最大权重(从最大权重节点开始递减选择)
                if (currentWeight == 0) {
                    return null;
                }
            }
        }
        if (servers.get(currentIndex).getWeight() >= currentWeight) {// 第一次:节点1~size权重>currentWeight(maxWeight),返回节点:最大权值
            return servers.get(currentIndex);
        }
    }
}

详见: http://zh.linuxvirtualserver.org/node/37
```
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-3-19/68971967.jpg)

- 最大公约数的意义
```
公约数在加权轮询中的意义: 在保持权重平衡(整数倍)的情况下,转换为更小值得表现(即减少"层高"),从而加速轮询周期(一圈)
最大公约数在加权轮询中的意义: 平衡基础的最小值表现,最快的轮询周期.(思考上图:仅调用6次情况下,有公约数处理可以提前达到负载均衡)
减最大公约数(gcd)相比每次减(1,2,最大公差,n)的优点:`维持整数比例关系`的`最小值表现`.公约数可保持权重平衡,最大公约数可最快的轮换节点.

关键字:`公约数保平衡`, `最大公约数得最小比例值`, `最大公约数做差可最快轮询周期,更少数量调用的均衡实现`
```

---
# 四.Zuul,Ribbon 源码分析
## 1.filter调用栈
- 入口servlet:  `ZuulController extends ServletWrappingController`
```
setServletClass(ZuulServlet.class);

return super.handleRequestInternal(request, response);
```
- ZuulServlet: com.netflix.zuul.http.ZuulServlet#service
```
    @Override
    public void service(javax.servlet.ServletRequest servletRequest, javax.servlet.ServletResponse servletResponse) throws ServletException, IOException {
        try {
            init((HttpServletRequest) servletRequest, (HttpServletResponse) servletResponse);

            // Marks this request as having passed through the "Zuul engine", as opposed to servlets
            // explicitly bound in web.xml, for which requests will not have the same data attached
            RequestContext context = RequestContext.getCurrentContext();
            context.setZuulEngineRan();

            try {
                preRoute();
            } catch (ZuulException e) {
                error(e);
                postRoute();
                return;
            }
            try {
                route();
            } catch (ZuulException e) {
                error(e);
                postRoute();
                return;
            }
            try {
                postRoute();
            } catch (ZuulException e) {
                error(e);
                return;
            }

        } catch (Throwable e) {
            error(new ZuulException(e, 500, "UNHANDLED_EXCEPTION_" + e.getClass().getName()));
        } finally {
            RequestContext.getCurrentContext().unset();
        }
```

- FilterProcessor: com.netflix.zuul.FilterProcessor#runFilters
```
public Object runFilters(String sType) throws Throwable {
    if (RequestContext.getCurrentContext().debugRouting()) {
        Debug.addRoutingDebug("Invoking {" + sType + "} type filters");
    }
    boolean bResult = false;
    List<ZuulFilter> list = FilterLoader.getInstance().getFiltersByType(sType);
    if (list != null) {
        for (int i = 0; i < list.size(); i++) {
            ZuulFilter zuulFilter = list.get(i);
            Object result = processZuulFilter(zuulFilter);
            if (result != null && result instanceof Boolean) {
                bResult |= ((Boolean) result);
            }
        }
    }
    return bResult;
}
```
- ZuulFilter: com.netflix.zuul.ZuulFilter#runFilter
```
public ZuulFilterResult runFilter() {
    ZuulFilterResult zr = new ZuulFilterResult();
    if (!isFilterDisabled()) {
        if (shouldFilter()) {
            Tracer t = TracerFactory.instance().startMicroTracer("ZUUL::" + this.getClass().getSimpleName());
            try {
                Object res = run();
                zr = new ZuulFilterResult(res, ExecutionStatus.SUCCESS);
            } catch (Throwable e) {
                t.setName("ZUUL::" + this.getClass().getSimpleName() + " failed");
                zr = new ZuulFilterResult(ExecutionStatus.FAILED);
                zr.setException(e);
            } finally {
                t.stopAndLog();
            }
        } else {
            zr = new ZuulFilterResult(ExecutionStatus.SKIPPED);
        }
    }
    return zr;
}
```
- PreHeaderFilter (自定义实现filter)
```
public class PreHeaderFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE; // 可以在请求被路由之前调用
    }

    @Override
    public int filterOrder() {
        return FilterConstants.DEBUG_FILTER_ORDER+1;
    }

    @Override
    public boolean shouldFilter() {
        return true;// 是否执行该过滤器，此处为true，说明需要过滤
    }

    @Override
    public Object run() {
        return null;
    }

}
```
- 至此,以上调用栈可以通过自有filter断点卡住,观察线程栈.

## 2.Route路由控制(forward)
### 情况一: route直接关系host
```
zuul.routes.riskService.stripPrefix=false
zuul.routes.riskService.path=/riskService/**
zuul.routes.riskService.routeHost=localhost:8090
```
- 进入SimpleHostRoutingFilter
```
public class SimpleHostRoutingFilter extends ZuulFilter {

    @Override
    public boolean shouldFilter() {
        return RequestContext.getCurrentContext().getRouteHost() != null
                && RequestContext.getCurrentContext().sendZuulResponse();
    }

    @Override
    public Object run() {
         ...
            CloseableHttpResponse response = forward(this.httpClient, verb, uri, request,
                    headers, params, requestEntity);
         ...
    }
}
```

### 情况二: route关系到serviceId(类集群)
```
zuul.routes.riskService.stripPrefix=false
zuul.routes.riskService.path=/riskService/**
zuul.routes.riskService.serviceId=originService
```
- 进入RibbonRoutingFilter
```
public class RibbonRoutingFilter extends ZuulFilter {

    @Override
    public boolean shouldFilter() {
        RequestContext ctx = RequestContext.getCurrentContext();
        return (ctx.getRouteHost() == null && ctx.get(SERVICE_ID_KEY) != null && ctx.sendZuulResponse());
    }

    @Override
    public Object run() {
         ...
         RibbonCommandContext commandContext = buildCommandContext(context);
         ClientHttpResponse response = forward(commandContext);
         ...
    }

}
```
- forward中还有`自定义负载均衡实现`. (怎么过来的,断点了解)
```
public class CustomWeightedResponseTimeRule extends RoundRobinRule {
    @Override
    public Server choose(ILoadBalancer lb, Object key) {
         ...
    }
}
```

---
## 3.Ribbon结构
### 核心类: BaseLoadBalancer 默认实现
```
IClientConfig ribbonClientConfig: DefaultClientConfigImpl配置
IRule ribbonRule: RoundRobinRule 轮询策略
IPing ribbonPing: DummyPing
ServerList ribbonServerList: ConfigurationBasedServerList
ServerListFilter ribbonServerListFilter: ZonePreferenceServerListFilter
ILoadBalancer ribbonLoadBalancer: ZoneAwareLoadBalancer
```

### 配置及可选项
```
NFLoadBalancerClassName
NFLoadBalancerRuleClassName
NFLoadBalancerPingClassName
NIWSServerListClassName
NIWSServerListFilterClassName
```

### NFLoadBalancerRuleClassName可选项(负载均衡)
```
# BestAvailableRule ：
最佳使用性规则，选择正在请求中的并发数最小的那个server，除非这个server在熔断中。

# ZoneAvoidanceRule：
区域敏感性规则，如果这个ip区域内有一个或多个实例不可达或响应变慢，都会降低该ip区域内其他ip被选中的权重。

# AvailabilityFilteringRule：
可用性敏感规则，首先轮询选择一个server，如果该server没有熔断并且正在请求中的数量没有达到限制，则选中它。

# WeightedResponseTimeRule：
响应平均值规则，起始为轮训算法，并开启一个计时器，每三十秒收集一次每个server的平均响应时间，当信息足够时，给每个server附上一个权重，并按权重随机选择server，高权重的server会被高概率选中。

# ConsistentHashRule：
hash一致性规则，如果http请求的header中存在一个key['rest_consistent_key']，则按它的value进行一致性hash选择相同的那个server，如果不存在，则使用服务启动时随机生成的一个字符串作为key。
```

### Pinger 工作机制(可作为健康检查)
`BaseLoadBalancer.PingTask` 10s的自动检测 / 每30s的service刷新检查
```
PingUrl 真实的去ping 某个url，判断其是否alive
PingConstant 固定返回某服务是否可用，默认返回true，即可用
NoOpPing 不去ping,直接返回true,即可用。
DummyPing 直接返回true，并实现了initWithNiwsConfig方法。
NIWSDiscoveryPing，根据DiscoveryEnabledServer的InstanceInfo的InstanceStatus去判断，如果为InstanceStatus.UP，则为可用，否则不可用。
```

### 动态创建Ribbon client
```
// 不存在时会自动创建默认Client
IClientConfig iClientConfig = springClientFactory.getClientConfig(serviceName);
// IClientConfig iClientConfig = ClientFactory.getNamedConfig(serviceName);
logger.debug("initWithNiwsConfig [{}] before:{}", serviceName, iClientConfig);

String loadBalancer = "com.netflix.loadbalancer.DynamicServerListLoadBalancer";
String rule = "cn.fraudmetrix.forseti.gateway.rule.CustomWeightedResponseTimeRule";// 自定义权重轮询

// 方式一:不可覆盖设置
// iClientConfig.set(CommonClientConfigKey.NFLoadBalancerClassName, loadBalancer);
// iClientConfig.set(CommonClientConfigKey.ListOfServers, listOfServers);
// iClientConfig.set(CommonClientConfigKey.NFLoadBalancerRuleClassName, rule);// 加权轮训

// 方式二:可以覆盖设置
ConcurrentCompositeConfiguration config = ((ConcurrentCompositeConfiguration) ConfigurationManager.getConfigInstance());
config.setOverrideProperty(serviceName + ".ribbon.NFLoadBalancerClassName", loadBalancer);// TODO 配置生效,实例未生效(暂无影响)
config.setOverrideProperty(serviceName + ".ribbon.listOfServers", String.join(",", listOfServers));
config.setOverrideProperty(serviceName + ".ribbon.NFLoadBalancerRuleClassName", rule);// 负载均衡规则:加权轮训
config.setOverrideProperty(serviceName + ".ribbon.ConnectTimeout", 1000);// 留意稳定性
config.setOverrideProperty(serviceName + ".ribbon.ReadTimeout", 60000);//2000

DynamicServerListLoadBalancer dynamicServerListLoadBalancer = (DynamicServerListLoadBalancer) springClientFactory.getLoadBalancer(serviceName);// 运行实例
dynamicServerListLoadBalancer.initWithNiwsConfig(dynamicServerListLoadBalancer.getClientConfig());
```

### 基础轮询算法
- RoundRobinRule
```
private AtomicInteger nextServerCyclicCounter;

/**
    * Inspired by the implementation of {@link AtomicInteger#incrementAndGet()}.
    *
    * @param modulo The modulo to bound the value of the counter.
    * @return The next value.
    */
private int incrementAndGetModulo(int modulo) {
    for (;;) {
        int current = nextServerCyclicCounter.get();
        int next = (current + 1) % modulo;
        if (nextServerCyclicCounter.compareAndSet(current, next))
            return next;
    }
}
```
- RoundRobinRuleTest
```
import java.util.concurrent.atomic.AtomicInteger;
public class RoundRobinRuleTest {

    private static AtomicInteger nextServerCyclicCounter;

    /**
     * Inspired by the implementation of {@link AtomicInteger#incrementAndGet()}.
     *
     * @param modulo The modulo to bound the value of the counter.
     * @return The next value.
     */
    private static int incrementAndGetModulo(int modulo) {
        for (; ;) {
            int current = nextServerCyclicCounter.get();
            int next = (current + 1) % modulo;
            if (nextServerCyclicCounter.compareAndSet(current, next))
                return next;
        }
    }
    public static void main(String[] args) {
        nextServerCyclicCounter = new AtomicInteger(0);
        for (int i = 0; i < 100; i++) {
            System.out.println(incrementAndGetModulo(3));
        }
    }
}
```

---
**参考**
`Getting Started · Netflix/ribbon Wiki · GitHub`
https://github.com/Netflix/ribbon/wiki/Getting-Started

`使用负载平衡器·Netflix / Ribbon Wiki·GitHub`
https://github.com/Netflix/ribbon/wiki/Working-with-load-balancers

`端到端示例·Netflix / ribbon维基·GitHub`
https://github.com/Netflix/ribbon/wiki/End-to-End-Examples

`深入理解Ribbon之源码解析`
http://blog.csdn.net/forezp/article/details/74820899

---
# 五. 工具 & 监控 (metrics采样,grafana制图及报警)
## 工具
- 网关基础状态观察页(gateway工作状态,故障节点情况)
    - http://localhost:8080/watchNowRoute
    - http://localhost:8080/watchNowServiceSpring
    - http://localhost:8080/cache
    - http://localhost:8080/cache/refresh
    - http://localhost:8080/cache/setClusterNodeWeighted?cluster=originService&server=localhost:8090&weighted=150 

## 监控 (metrics采样,grafana制图及报警)
详见: http://10.57.17.82:3000/dashboard/db/forseti-gateway?orgId=1&from=now-30m&to=now&var-interval=10s
```
forseti.gateway.cluster
forseti.gateway.count
forseti.gateway.filter
forseti.gateway.zuul
forseti.gateway.exception
```
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-3-19/6378748.jpg)

---
# 六.指标情况(基准测试) 
## 压测结果
local测试: forseti-gateway rt : 1~10ms. 平均2ms

TODO 待压测出详细指标...

...

## 网摘(仅供参考) : 绩效基准汇总如下
1.Micro - 单核CPU情况下：
```
（1）直接访问：每秒6519.68个请求，每个请求花费时间30.676ms
（2）Nginx：每秒4888.24个请求, 每个请求花费时间40.915ms
（3）Zuul: 每秒950.57个请求, 每个请求花费时间210.399ms
```
2.在M4.Large - 双核CPU情况下：
```
（1）Nginx：每秒6187.14个请求，每个请求花费时间32.325ms
（2）Zuul: 每秒2099.93个请求，每个请求花费时间95.241ms
```
3.在M4.2xLarge - 8核CPU情况下：
```
（1）zuul:每秒7036.9个请求，每个请求花费时间28.422ms
（2）Linkerd: 每秒6995个请求，每个请求花费时间28.592ms
（3）Nginx: 每秒6233.4个请求，每个请求花费时间32.085ms
（4）Spring Cloud Gateway：每秒873.14个请求，每个请求花费时间229.058ms
```
Spring Cloud Gateway每秒可以处理873个请求，每个请求的平均时间为229ms。根据我们的测试，Spring Cloud Gateway的性能无法达到Zuul，Linkerd和Nginx的水平，至少在Github的当前代码库就是这种情况。

**参考**
`微服务API网关NGINX、ZUUL、Spring Cloud Gateway与 - 推酷`
https://www.tuicool.com/articles/Fbu26jM

`Netflix Zuul与Nginx 性能对比` - Zuul的原始性能非常接近于Nginx
https://www.tuicool.com/articles/ErEVZf6
