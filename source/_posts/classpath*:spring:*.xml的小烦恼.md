title: SpringJUnit, classpath*:spring/*.xml 的小烦恼 & SpringJunit与Mock怎么选?
date: 2018-8-25 00:00:00
categories: junit
tags: [junit]



---

[TOC]


---

# 一.问题:初始化Bean`esNotifyConfig`依赖的Bean`ElasticSearchNotifyConfig`无法注入(@Autowired)

```

Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'esSwitchInitialState': Unsatisfied dependency expressed through field 'esNotifyConfig'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'cn.***.module.elasticsearch.basic.ElasticSearchNotifyConfig' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}

```

## 定位问题Bean

```

@Service("esSwitchInitialState")
public class SwitchStateFromZk {

    @Autowired
    private ElasticSearchNotifyConfig esNotifyConfig;

}



-- 日志信息:`PathMatchingResourcePatternResolver [cn/***/module/elasticsearch/]`

PathMatchingResourcePatternResolver - Resolved classpath location [cn/***/module/elasticsearch/] to resources [URL [jar:file:...]]
PathMatchingResourcePatternResolver - Looking for matching resources in jar file [file:/Users/liuxiang/.m2/repository/cn/***/module-elasticsearch/5.0.45/module-elasticsearch-5.0.45.jar]
PathMatchingResourcePatternResolver - Resolved location pattern [classpath*:...]]

```

## 疑问:哪里扫描的这个路径(cn/***/module/elasticsearch/)?

- ***/module-elasticsearch-5.0.45.jar!/spring/module-elasticsearch.xml
```
<context:component-scan base-package="cn.***.module.elasticsearch"/>
```

## 为什么会去扫描jar库下的路径?

- 日志中找到线索:`[classpath*:spring/*.xml]`时扫描到了大量的jar包中的`spring/***.xml` (其中依赖的Bean全部会被加载)`

---

# 二.怎么办?

## 方法一: 取消jar依赖,避免被`[classpath*:spring/*.xml]`扫描到而加载

```
<dependency>
    <groupId>cn.***</groupId>
    <artifactId>creditcloud-share</artifactId>
    <exclusions>
        <exclusion>
            <groupId>cn.***</groupId>
            <artifactId>module-elasticsearch</artifactId>
        </exclusion>
    </exclusions>
</dependency>

```



## 方法二: jar库提供方(creditcloud-share)需控制好依赖,仅打包相关的mudel.  (`★★`)
- 如 `**-dubbo-client.jar` 包含: interface,DataBean,Spring(注册Dubbo consumer)-可选. 其它无关的尽量不要打入包中.

## 方法三: 使用方不要盲目使用`[classpath*:spring/*.xml]`,容易因外部jar库的影响.  (常见于单测) (`★★★`)
```

@RunWith(SpringJUnit4ClassRunner.class)
// @ContextConfiguration(locations = {"classpath*:spring/*.xml"})  // 会自动扫描到外部jar库中的`spring/`路径.容易受外部依赖的影响

@ContextConfiguration(locations = {"classpath:spring/*.xml"})// 仅扫描当前mudul下`spring/`

```


---

# 三.另一个话题: 单测 SpringJunit 与 Mock (JMockit / PowerMock / Mockito...) 怎么选?
## Spring容器状态测试
- 优:
    - 接近服务运行状态(Spring),结合入口测试,能方便的覆盖大量代码.
    - 能覆盖到数据源的操作,并自动回滚事务
    - 弥补mock方式需预先mock依赖Bean的繁琐,相关依赖由Spring管理
- 缺: 
    - 启动效率相对Mock慢台多. (相对适合轻量容器的应用)
    - 不同分支的覆盖,可能需要从更早的入口开始测试.相比mock更累赘
    - 异常的覆盖,相关mock更难构建
    - 单测的结果受数据源的影响:如清理数据库时,大量单测有可能会失败
    - 在一个事务中,完成多场景的测试较为困难.不控制事务又将污染数据源
    - 启动依赖(bean初始化)不满足时,所有的单测全部失败
    - test下spring/**.xml 需要与main同时维护,否则单测会频频失败
- 适: 更关心外部件(消息,存储)的关系测试. 及更多mock方式不擅长的场景

## Mock对象状态测试

- 优:
    - 启动轻快
    - 自由mock对象,出参,异常.能更方便覆盖方法的每个角落
    - 单测结果稳定,不受数据源影响
- 缺:
    - 被测目标,相关依赖Bean,均需提前设定mock,较为繁琐
    - 存在部分对象无法mock,导致代码很难覆盖
    - 对new,private,static情况,相对麻烦一些
    - 对长链路覆盖不是很方便,一般只对单方法覆盖及同类的关系方法
- 适: 关心更小单元逻辑测试 / 新增代码的单测覆盖