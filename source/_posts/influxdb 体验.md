title: influxdb 体验
date: 2018-2-23 00:00:00
categories: influxdb
tags: [influxdb]

---

# 介绍
InfluxDB是time-series data数据库，负责高效处理实时数据。

`InfluxData资源| 案例研究，网络研讨会，技术论文，视频`
https://www.influxdata.com/_resources/
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/18-4-28/31590231.jpg)

---
# 方法一：使用influxdb客户端查询数据
influxdb自带一个比较完备的命令行客户端，可以通过使用influxQL进行数据查询。
0. 通过下载我提供的客户端: `influx`
1. 通过homebrew安装:(默认你已经安装了homebrew): `brew install influxdb`
2. 通过go直接安装:`go install github.com/influxdata/influxdb`
3. 使用influx命令
在安装完influxdb之后你其实获得了两个命令，一个influxd，是influxdb的server；一个influx，是influxdb的client。
使用如下命令链接influxdb
````
influx -host 10.57.17.82 -port 8086
```

# 方法二：通过InfluxDB Admin Interface
influxdb启动后默认会起动一个influxdb admin interface的web服务，默认端口为8083。比如访问线下的influxdb的influxdb admin interface
http://10.57.17.82:8083
默认是连接上了当前的influxdb。

# influxdb的一些常用查询语句
切换数据库:`use forseti-api`
查看当前数据库的measurement: `show measurements`
查看当前数据库的series:`show series`
查找最近一分钟的数据:`select * from "creditcloud.apply.consumer" where time > now() - 1m limit 10`
按天统计调用量:`select sum(value) from score_scard_qps where time > now() - 10d group by time(1d)`

注意measurement名字中间有"."的一定要用引号包裹起来。
请务必在查询的时候限定时间区间，influxdb挂了可是要赔钱的。
其他命令参考: https://docs.influxdata.com/influxdb/v1.0/

`如何查询Influxdb的数据 - 同盾研发 - 同盾科技-Confluence`
http://wiki.tongdun.me/pages/viewpage.action?pageId=14747599

---
# 检查influxdb中写入的数据
如果是开发环境，可以直接前往 http://10.57.17.82:8083/ 输入查询语句查看数据。或者通过influxdb的http api进行查询。
#列出所有database 语句 show databases
```
curl "http://10.57.17.82:8086/query?db=testdb&q=show%20databases"
```

#列出所有measurement 语句 show measurements
````
curl "http://10.57.17.82:8086/query?db=testdb&q=show%20measurements"
```

#查询一个measurement最近10s内的数据 
```
select * from "Demo.execute" where time > now() - 10s 名字里带点的要用引号
```
```
curl "http://10.57.17.82:8086/query?db=testdb&q=select%20*%20from%20%22Demo.execute%22%20where%20time%20%3E%20now()%20-%2010s"
```

`Influxdb性能监控接入以及Grafana可视化展示 - 徐圣 - 同盾科技-Confluence`
http://wiki.tongdun.me/pages/viewpage.action?pageId=9151299

---
# 记录内容
- 示例: module-metrics-1.2.19.jar (详见:http://wiki.tongdun.me/pages/viewpage.action?pageId=19403620)
关键:上报周期(5s)区间内,会对相同tag组合,会进行`组合计算(rt去平均,qps取总,count取总)`
约定:不能把`sequenceId`加入tag组合,这样会导致上报区间数据,无法`组合计算`,导致数据量大增.
```
{
    "results": [
        {
            "statement_id": 0,
            "series": [
                {
                    "name": "forseti.api.filter",
                    "columns": [
                        "time", -- 上报时间,非每次调用的时间.
                        "cluster", -- tag
                        "filter", -- tag
                        "host", -- tag
                        "value" -- 耗时统计: 对上报周期(5s)区间内,会对相同tag组合,会进行`组合计算(rt去平均,qps取总,count取总)`
                    ],
                    "values": [
                        [
                            "2018-03-07T02:27:31.287Z",
                            "unknown",
                            "BasicDataFilterImpl",
                            "10.57.17.47",
                            5
                        ],
                        [
                            "2018-03-07T02:27:31.287Z",
                            "unknown",
                            "FlowCharageFilterImpl",
                            "10.57.17.47",
                            0
                        ],
                        ...
```

---
# 删除表(measurement)
```
DROP MEASUREMENT <measurement_name>
show measurements

如:
drop measurement "forseti.gateway.count"
drop measurement "forseti.gateway.filter"
drop measurement "forseti.gateway.zuul"
show measurements
```
详见: https://docs.influxdata.com/influxdb/v1.1/query_language/database_management/#delete-measurements-with-drop-measurement

---
**参考**
`influxdata监控系统简介 - CSDN博客`
http://blog.csdn.net/james_wade63/article/details/50838201

`使用influx控制台工具操作InfluxDB - 推酷`
https://www.tuicool.com/articles/jemAneM

`InfluxData | Documentation | Comparison to SQL`
http://docs.influxdata.com/influxdb/v0.9/concepts/crosswalk/#influxql-and-sql