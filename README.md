# 基础使用
- 编译 hexo g
- 服务 hexo s
- 发布 hexo d

---
# 提交
```
git status
git add .
git commit -am 'update'
git push
```

---
# 更新blog
## 方式一: travis-ci 监听push,自动更新
- 配置:.travis.yml
- 在线可见过程 https://travis-ci.org/

## 方式二: hexo d 发布
- 配置:_config.yml中deploy

详见:﻿blog工具选择 & travis-ci自动构建.md
