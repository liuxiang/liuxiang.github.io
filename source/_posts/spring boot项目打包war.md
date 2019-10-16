title: spring boot项目打包war
date: 2017-12-22 00:00:00
categories: spring boot
tags: [spring boot]

---
[TOC]

---
# spring boot编译war包,支持预设
```
@SpringBootApplication
@ComponentScan(basePackages = "me.liuxiang")
@ImportResource(locations = {"classpath:app.xml"})
public class AppApplication extends SpringBootServletInitializer{
    ...
}
```

---
# 一.编译war包
## 1.Idea编译(service:war exploded)
- target/chick-service-0.0.1-SNAPSHOT/META-INF/MANIFEST.MF  - `外置tomcat不可运行`
```
Manifest-Version: 1.0
Implementation-Version: 0.0.1-SNAPSHOT
Built-By: liuxiang
Implementation-Vendor-Id: me.liuxiang
Created-By: IntelliJ IDEA
Build-Jdk: 1.8.0_141
Main-Class: ${start-class}
```

## 2.增加main-class(Idea编译) 
- pom.xml
```
<properties>
    <!-- war启动类 -->
    <start-class>me.liuxiang.chick.AppApplication</start-class>
</properties>
```
- MANIFEST.MF - `外置tomcat可以运行`
```
Manifest-Version: 1.0
Implementation-Version: 0.0.1-SNAPSHOT
Built-By: liuxiang
Implementation-Vendor-Id: me.liuxiang
Created-By: IntelliJ IDEA
Build-Jdk: 1.8.0_141
Main-Class: me.liuxiang.chick.AppApplication
```

---
## 3.maven编译
### 未设置<build><plugins>插件,无法编译
### 增加`maven-war-plugin
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <failOnMissingWebXml>false</failOnMissingWebXml>
    </configuration>
</plugin>
```
效果: cash-service-1.0-sources.jar & cash-service-1.0.jar  - `外置tomcat不可运行`
```
Manifest-Version: 1.0
Archiver-Version: Plexus Archiver
Created-By: Apache Maven
Built-By: liuxiang
Build-Jdk: 1.8.0_141
```
### 增加`spring-boot-maven-plugin`
```
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <!--configuration>
        <fork>true</fork>
    </configuration-->
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
效果: cash-service-1.0.jar  - `外置tomcat可以运行`
```
Manifest-Version: 1.0
Archiver-Version: Plexus Archiver
Built-By: liuxiang
Start-Class: cn.***.cash.service.Application
Spring-Boot-Classes: WEB-INF/classes/
Spring-Boot-Lib: WEB-INF/lib/
Spring-Boot-Version: 2.0.0.M7
Created-By: Apache Maven 3.5.0
Build-Jdk: 1.8.0_141
Main-Class: org.springframework.boot.loader.WarLauncher
```

---
# 二.排除内置`tomcat`
(不排除,main方式和war方式都可运行. 如排除仅war方式可运行)
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- 移除嵌入式tomcat插件 -->
    <!--<exclusions>-->
        <!--<exclusion>-->
            <!--<groupId>org.springframework.boot</groupId>-->
            <!--<artifactId>spring-boot-starter-tomcat</artifactId>-->
        <!--</exclusion>-->
    <!--</exclusions>-->
</dependency>
<!-- 这里指定打包的时候不再需要tomcat相关的包(会导致main方式启动失败) -->
<!--<dependency>-->
    <!--<groupId>org.springframework.boot</groupId>-->
    <!--<artifactId>spring-boot-starter-tomcat</artifactId>-->
    <!--<scope>provided</scope>-->
<!--</dependency>-->
```


---

**参考**
`spring boot打包war,tomcat运行 - 简书`
https://www.jianshu.com/p/5a0378129d0c

`SpringBoot初探（二）——打成war和自动部署 - CSDN博客`
http://blog.csdn.net/u010317202/article/details/50379057

`spring boot项目打包成war并在tomcat上运行的步骤`
http://blog.csdn.net/yalishadaa/article/details/70037846

---
**参考(启动原理)**
`spring boot应用启动原理分析 - ImportNew` - JarLauncher
http://www.importnew.com/19149.html

`SpringBoot应用启动流程 - 沙与沫 - 博客园` WarLauncher
https://www.cnblogs.com/lmk-sym/p/6554382.html