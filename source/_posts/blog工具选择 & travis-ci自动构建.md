title: blog 之 hexo&travis-ci
date: 2017-9-14 00:00:00
categories: hexo
tags: [hexo,travis-ci]

---
# 一.为什么要写blog
![](http://7xnbs3.com1.z0.glb.clouddn.com/15-12-18/30111185.jpg)

`我们为什么应该坚持写博客 - 推酷`
http://www.tuicool.com/articles/EjIV7nv

`书写是为了更好的思考 – 刘未鹏 | Mind Hacks`
http://mindhacks.cn/2009/02/09/writing-is-better-thinking/

`为什么你应该（从现在开始就）写博客 – 刘未鹏 | Mind Hacks`
http://mindhacks.cn/2009/02/15/why-you-should-start-blogging-now/

---
# 二.博客的选择
- 知识平台
    - 综合类：LOFTER，点点，`简书（推荐）`
    - 技术类：javaeye,博客园,csdn,oschina,SegmentFault
    - 优点：SEO友好，平台推广，无服务，markdown（`适合懒人`）
    - 缺点：个性受控，域名，无法比对更新
- CMS建站模板
    - WordPress,Webflow,Squarespace
    - 优点：个性选择，域名自建，管理更新 （`适合站长`）
    - 缺点：受限模板，非纯净的文章系统，无托管容器（云引擎php）
- 网站生成器
    - hexo,Jekyll,Hugo
    - 优点：个性，自定义，markdown，评论，比对更新（`适合码农`）
    - 缺点：本地构建（可通过ci托管改善）,寻托管容器（github pages）
- 新奇
    - github issue
    - 优点：markdown，状态，评论，邮件（`适合团队`）
    - 缺点：域名，无法比对更新

`前端开发中遇到的工程问题`https://github.com/fouber/blog/issues
`天猫前端`https://github.com/tmallfe/tmallfe.github.io/issues

`个人博客，最终选择。 · Issue #7 · atian25/blog`
https://github.com/atian25/blog/issues/7#issue-50490772

---
# 三.怎么写好文章

[中文技术文档的写作规范-阮一峰](http://www.ruanyifeng.com/blog/2016/10/document_style_guide.html)
[程序员怎样才能写出一篇好的技术文章 - 推酷](http://www.tuicool.com//articles/qa2ARb)
[你的文章改个标题也能轻松10万+ - 推酷](http://www.tuicool.com//articles/F3673aJ)

---
# 四.blog 之 hexo&travis-ci
## 1.概念
hexo：[静态博客生成器](https://www.staticgen.com)工具 （类：Jekyll,Hugo）
yaml：标记语言([配置文件](http://colobu.com/2017/08/31/configuration-file-format/?utm_source=tuicool&utm_medium=referral)) （类：json,xml,properties）
markdown：文档格式模板语言（类：HTML）

travis-ci：在线开源持续集成构建工具（类：jenkins）
github-pages：免费的静态资源部署容器（类：Coding Pages）

---
## 2.安装hexo
- 安装
```
$ npm install hexo-cli -g        #npm安装hexo-cli
$ hexo init blog            #hexo初始化,命名blog
$ cd blog                #进入目录
$ npm install                #安装依赖模块
$ hexo server                #开启预览(http://localhost:4000,Press Ctrl+C to stop)

$ hexo generate        #生成静态页面
$ hexo server        #开启预览(http://localhost:4000,Press Ctrl+C to stop)
```
- 模板选择
`有哪些好看的 Hexo 主题？ - 知乎` https://www.zhihu.com/question/24422335
(自助github搜：[`hexo-theme`](https://github.com/search?o=desc&q=hexo-theme&s=stars&type=Repositories&utf8=%E2%9C%93))

详见：
`hexo你的博客`  http://www.tuicool.com/articles/AfQnQjy/
[`手把手搭blog & github pages,hexo.md`](http://liuxiang.github.io/2016/03/13/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%90%ADblog%20&%20github%20pages,hexo/)

## 3.配置`travis-ci`
- 准备
    - github `.github.io`新建`hexo-source`分支(内容为hexo工程源码)
    - 根目录下新建`.travis.yml`,配置相关`travis-ci.org`构建申明
    - github中生成access token [Settings > Personal access tokens > 选择权限 > 生成token] (待travis-ci中提交代码授权使用)
- github登录 https://travis-ci.org/
- 新建Repositories > 选择`.github.io`项目
- travis中配置GH_TOKEN常量. [Settings > Environment Variables > key:GH_TOKEN value:github-token]
- 仓库`.github.io`代码发生代码push时，会自动触发travis-ci构建，其中包含将构建结果push到marster. 最终blog内容将自动更新。
![](http://7xnbs3.com1.z0.glb.clouddn.com/17-9-14/70710785.jpg)
 
`.travis.yml`
```
language: node_js  #设置语言

node_js: stable  #设置相应的版本

install:
  - npm install  #安装hexo及插件

script:
  - hexo cl  #清除
  - hexo g  #生成

after_script:
  - cd ./public
  - git init
  - git config user.name "yourname"  #修改name
  - git config user.email "your email"  #修改email
  - git add .
  - git commit -m "update"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master  #GH_TOKEN是在Travis中配置token的名称

branches:
  only:
    - hexo  #只监测hexo分支，hexo是我的分支的名称，可根据自己情况设置
env:
global:
  - GH_REF: github.com/yourname/yourname.github.io.git  #设置GH_REF，注意更改yourname
```

细节详见：http://www.jianshu.com/p/92dc63551b80

---
## 4.可能遇到的问题
### 问题一：`hexo g` 时 `Error: Unable to call `the return value of ***`
```
$ hexo g
INFO  Start processing
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Template render error: (unknown path) [Line 8, Column 23]
  Error: Unable to call `the return value of (posts["first"])["updated"]["toISOString"]`, which is undefined or falsey
    at Object.exports.prettifyError (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/hexo-generator-feed/node_modules/nunjucks/src/lib.js:34:15)
    at /home/travis/build/liuxiang/liuxiang.github.io/node_modules/hexo-generator-feed/node_modules/nunjucks/src/environment.js:489:31
    at new_cls.root [as rootRenderFunc] (eval at _compile (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/hexo-generator-feed/node_modules/nunjucks/src/environment.js:568:24), <anonymous>:210:3)
    at new_cls.render (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/hexo-generator-feed/node_modules/nunjucks/src/environment.js:482:15)
    at Hexo.module.exports (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/hexo-generator-feed/lib/generator.js:40:22)
    at Hexo.tryCatcher (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/bluebird/js/release/util.js:16:23)
    at Hexo.<anonymous> (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/bluebird/js/release/method.js:15:34)
    at /home/travis/build/liuxiang/liuxiang.github.io/node_modules/hexo/lib/hexo/index.js:340:24
    at tryCatcher (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/bluebird/js/release/util.js:16:23)
    at MappingPromiseArray._promiseFulfilled (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/bluebird/js/release/map.js:61:38)
    at MappingPromiseArray.PromiseArray._iterate (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/bluebird/js/release/promise_array.js:114:31)
    at MappingPromiseArray.init (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/bluebird/js/release/promise_array.js:78:10)
    at MappingPromiseArray._asyncInit (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/bluebird/js/release/map.js:30:10)
    at Async._drainQueue (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/bluebird/js/release/async.js:138:12)
    at Async._drainQueues (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/bluebird/js/release/async.js:143:10)
    at Immediate.Async.drainQueues (/home/travis/build/liuxiang/liuxiang.github.io/node_modules/bluebird/js/release/async.js:17:14)
```
- 原因：生成 RSS 也就是订阅的插件同 Hexo 3.2 的兼容性的问题
- 解决：停用不兼容插件plugins
```
# plugins:
#  - hexo-generator-feed
#  - hexo-generator-sitemap
```
http://www.cnblogs.com/qyun/p/6628601.html
https://hexo.io/docs/troubleshooting.html

### 问题一：`git push *`时`fatal: Authentication failed for 'https://@github.com/liuxiang/liuxiang.github.io.git/'`
```
The command "hexo g" exited with 0.
$ cd ./public
$ git init
Initialized empty Git repository in /home/travis/build/liuxiang/liuxiang.github.io/public/.git/
$ git config user.name "liuxiang"
$ git config user.email "liuxiang.1227@qq.com"
$ git add .
$ git commit -m "Update docs"
[master (root-commit) 3a6badf] Update docs
$ git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:master
remote: Anonymous access to liuxiang/liuxiang.github.io.git denied.
fatal: Authentication failed for 'https://@github.com/liuxiang/liuxiang.github.io.git/'
```
- 原因：${GH_TOKEN}常量没有配置
- 解决：
    - 1.先到github中生成access token。[Settings > Personal access tokens > 选择权限 > 生成token]
    - 2.再到travis中配置GH_TOKEN常量。[Settings > Environment Variables > key:GH_TOKEN value:github-token]
