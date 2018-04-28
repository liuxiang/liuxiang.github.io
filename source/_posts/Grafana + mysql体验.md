title: Grafana + mysql体验
date: 2018-2-5 00:00:00
categories: Grafana
tags: [Grafana]

---
[TOC]

---
# 介绍
在时序分析及监控展现领域，Grafana无疑是开源解决方案中的翘楚，其灵活的插件机制，支持各种漂亮的面板、丰富的数据源以及强大的应用。典型的面板有Graph、Text、Singlestat、PieChart、Table、Histogram等，支持的数据源有`ES`、Graphite、`InfluxDB`、OpenTSDB、`MySQL`、Druid 、`Prometheus`、`SimpleJson`等，提供的应用有Zabbix、K8s等

- +Elasticsearch
印象中Grafana是 InfluxDB 可视化工具，但基于Grafana良好的架构，DataSource可以变得多样性了，现在是支持ElasticSearch作为DataSource的。 Grafana的安装也非常简单，解压启动就可以了。DataSource都是动态添加上去的。Grafana作为一个纯粹的可视化工具，灵活性做到了极致。使用ElasticSearch作为DataSource，指定对应的Index，就可以建立DashBoard就行数据可视化了。相比Kibana，Grafana的template功能能把数据按照任意的维度进行切分展示，这就是它的强大之处。

---
# 部署
Github地址：https://github.com/Grafana/Grafana

- 下载
https://grafana.com/grafana/download

- mac安装
```
brew update 
brew install grafana
```
http://docs.grafana.org/installation/mac/
http://docs.grafana.org/installation/configuration/

- 启动Grafana 
```
要启动Grafana使用自制服务首先确保安装了自制软件/服务。
brew tap homebrew/services

然后开始Grafana使用：
brew services start grafana
- 配置
配置文件应该位于/usr/local/etc/grafana/grafana.ini。
- 日志
日志文件应该位于/usr/local/var/log/grafana/grafana.log。
- 插件
如果你想手动安装一个插件放在这里：/usr/local/var/lib/grafana/plugins。
- 数据库
默认的sqlite数据库位于 /usr/local/var/lib/grafana
```
- 汉化(非必要)
```
git clone https://github.com/moonstack/grafana-for-chinese.git
备份原有的 /public/app/boot.js /public/app/features/org/prefs_control.js 再将master中的两个文件覆盖到对应的目录 刷新浏览器即可
```
https://github.com/moonstack/grafana-for-chinese
https://github.com/heruihong/gf-frontend

- 访问
http://0.0.0.0:3000/ ，默认账号/密码：admin/admin

- 添加数据源
http://docs.grafana.org/features/datasources/elasticsearch/
http://docs.grafana.org/features/datasources/mysql/

- 导入dashboard/查看dashboard
https://grafana.com/dashboards?dataSource=mysql
https://grafana.com/dashboards?dataSource=elasticsearch

---
# 制图(mysql)
`Grafana + mysql数据源 - laomei - CSDN博客`
http://blog.csdn.net/sweatott/article/details/78278011

## 图表数据展示规则 - `edit - Metrics`
```
SELECT
  DATE_FORMAT(gmt_create,'%Y-%m-%d'),
  UNIX_TIMESTAMP(DATE_FORMAT(gmt_create,'%Y-%m-%d')) as time_sec, -- x轴:时间
  count(treaty_no) as value, -- y轴:数值
  partner_code as metric -- y轴分组:数据分组
FROM treaty
WHERE 1=1
group by DATE_FORMAT(gmt_create,'%Y-%m-%d')
ORDER BY id ASC
```
更多文档见:http://docs.grafana.org/features/datasources/mysql/
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-2-8/29820002.jpg)

## 各报表面板设置
http://docs.grafana.org/features/panels/graph/ `图表`
http://docs.grafana.org/features/panels/singlestat/ `仪图`

## templating 模板
http://docs.grafana.org/reference/templating/
http://docs.grafana.org/features/datasources/mysql/#query-variable `mysql的查询表达式`
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-2-8/86842944.jpg)

![](http://7xnbs3.com1.z0.glb.clouddn.com/18-3-19/55292517.jpg)
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-3-19/45274927.jpg)

- 在查询中使用变量
http://docs.grafana.org/features/datasources/mysql/#using-variables-in-queries
```
SELECT
  DATE_FORMAT(gmt_create,'%Y-%m-%d'),
  UNIX_TIMESTAMP(DATE_FORMAT(gmt_create,'%Y-%m-%d')) as time_sec, -- x轴:时间
  count(treaty_no) as value, -- y轴:数值
  partner_code as metric -- y轴分组:数据分组
FROM treaty
WHERE 1=1 and partner_code in ($partner)
group by DATE_FORMAT(gmt_create,'%Y-%m-%d')
ORDER BY id ASC
```
- 效果
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-2-8/52927063.jpg)

---
# 报警
http://docs.grafana.org/alerting/notifications/
![](http://7xnbs3.com1.z0.glb.clouddn.com/18-2-8/8688942.jpg)

- Grafana添加`Webhook channel`
https://www.tuicool.com/articles/VZf6JjF

- [图表-alert]设置数据触警规则
-  [图表-alert]Notifications 定义发送渠道

---
# 问题
## 安装grafana,Homebrew出现"go: version missing for "gotools" resource!"的解决方案
```
➜ Desktop brew install grafana
Error: go: version missing for "gotools" resource!
```
尝试解决:
```
git -C "$(brew --repo)" fetch --tags
brew update --force
```
或 `git -C "$(brew --repo)" fetch --tags brew update --force`
http://blog.csdn.net/lanadeus/article/details/78271240
https://github.com/Homebrew/homebrew-core/issues/19221
https://blog.labbit.jp/%E8%AC%8E%E3%82%A8%E3%83%A9%E3%83%BCgotools%E3%81%AE%E8%A7%A3%E6%B6%88%E6%B3%95/

实际解决: 重装brew 
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## 新建template query 错误
```
Templating
Template variables could not be initialized: sql: Scan error on column index 0: unsupported Scan, storing driver.Value type <nil> into type *string
```
原因: 存在null值
解决: `where column is not null`


---
**参考**
`基于Grafana+SimpleJson的灵活报表解决方案`
https://www.tuicool.com/articles/EBjUfmi

`监控数据的可视化分析神器 Grafana - 推酷`
https://www.tuicool.com/articles/VZf6JjF

`Kibana and Grafana - 推酷`
https://www.tuicool.com/articles/rURRF3U

`使用Prometheus+Grafana搭建监控系统实践 - 推酷`
https://www.tuicool.com/articles/QZRjAv3

`监控数据的可视化分析神器 Grafana - 推酷`  - `OneAlert 云告警`
https://www.tuicool.com/articles/VZf6JjF

`grafana + influxdb + telegraf , 构建linux性能监控平台 - 推酷`
https://www.tuicool.com/articles/nyM3qyq
