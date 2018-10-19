title: Kibana + Elasticsearch体验
date: 2018-2-5 00:00:00
categories: Kibana
tags: [Kibana]

---
[TOC]

---
# 安装kibana
## 下载 
https://www.elastic.co/downloads/kibana
`wget https://artifacts.elastic.co/downloads/kibana/kibana-6.1.3-darwin-x86_64.tar.gz`

## 准备elasticsearch服务
见: Elasticsearch相关.md
- Elasticsearch
Kibana安装非常方便，官网下载，并配置下ES的URL就可以连上了。Kibana使用的时候会有副作用，它会在你的ES里面创建一个它所需要的Index，并会有较为频繁的query。如果是生产环境，ES存的是核心数据要慎用了。安装好启动指定需要分析的Index，就能看到以下几个Tab


## 配置 config/kibana.yml
```
server.port: 5601

# The host to bind the server to.
server.host: ""

# If you are running kibana behind a proxy, and want to mount it at a path,
# specify that path here. The basePath can't end in a slash.
# server.basePath: ""

# The Elasticsearch instance to use for all your queries.
elasticsearch.url: "http://0.0.0.0:9200" #这里是elasticsearch的访问地址
```

## 汉化 
https://github.com/anbai-inc/Kibana_Hanization
`python main.py <kibana目录>`

## 启动
```
./kibana //不能关闭终端
nohup ./kibana > /nohub.out & //可关闭终端，在nohup.out中查看log
```

## 访问 http://0.0.0.0:5601/

---
# 使用
## 新建索引模式
http://0.0.0.0:5601/app/kibana#/management/kibana/indices

## 新建图表&设置xy轴数据列
http://0.0.0.0:5601/app/kibana#/visualize/new?_g=()
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/18-2-8/98836357.jpg)

## 仪表盘集成图表
![](http://ll-blog.oss-cn-hangzhou.aliyuncs.com/18-2-8/81140044.jpg)

---
**参考**
`Kibana and Grafana - 推酷`
https://www.tuicool.com/articles/rURRF3U

`kibana介绍 - CSDN博客`
http://blog.csdn.net/eff666/article/details/63251710?locationNum=3&fps=1