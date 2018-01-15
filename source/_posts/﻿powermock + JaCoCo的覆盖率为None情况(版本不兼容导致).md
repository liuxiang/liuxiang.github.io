title: powermock + JaCoCo的覆盖率为None情况(版本不兼容导致)
date: 2017-12-28 00:00:00
categories: powermock
tags: [powermock]

---
# powermock + JaCoCo组合,官方说明
```
JaCoCo Offline Instrumentation works only with PowerMock version `1.6.6 and above`.
即要求: PowerMock升级到1.6.6+
```
https://github.com/powermock/powermock/wiki/Code-coverage-with-JaCoCo

# pom.xml建议
> 注:1.6.6+ 不一定兼容低版本单测代码.
```
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-all</artifactId>
    <!--<version>1.9.5</version>-->
    <version>1.10.19</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <!--<version>1.5.6</version>-->
    <version>1.6.6</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito</artifactId>
    <!--<version>1.5.6</version>-->
    <version>1.6.6</version>
    <scope>test</scope>
</dependency>
```

---
**参考**
`Powermock and sonar jacoco的覆盖率不兼容问题解决 3`
http://blog.csdn.net/cloud_ll/article/details/52194399

`对于用Powermock编写的测试用例，sonar中单元测试覆盖率统计不正确的问题`
http://blog.csdn.net/qq_23589557/article/details/68938858

`wiki/PowerMockRule`
https://github.com/powermock/powermock/wiki/PowerMockRule

`PowerMock is of no use for code coverage #509`
https://github.com/powermock/powermock/issues/509
