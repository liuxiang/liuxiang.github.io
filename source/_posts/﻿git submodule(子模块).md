title: git submodule(子模块)
date: 2017-12-08 00:00:00
categories: git
tags: [git]

---
[TOC]

---
# 常用命令
```
git clone <repository> --recursive 递归的方式克隆整个项目
git submodule add <repository> <path> 添加子模块
git submodule init 初始化子模块
git submodule update 更新子模块
git submodule foreach git pull 拉取所有子模块
```
---
# 体验
- git主模块无法对`子模块(submodule)`,进行提交,更新等管理
- 子模块修改,主模块`git status`可察觉,但无法提交.需进入模块中提交,再回到主模块更新`git submodule update`
- 适合场景:子模块只为依赖 (对子模块内容只做更新,不做修改)
- 如是`维护型`子模块,不建议使用`git子模块`,建议本地利用`Maven/IDE的子模块`直接维护,还能感知变化和维护.

---
# 添加
```
git submodule add https://github.com/**.git
```

# 差异比较
```
git diff --cached --submodule
```

# clone
```
git clone https://github.com/chaconinc/MainProject
...
git submodule init 
git submodule update
```
或(`--recursive`)
```
git clone --recursive https://github.com/chaconinc/MainProject
```

# 更新
```
git submodule update --remote DbConnector
```
或
```
git config -f .gitmodules submodule.DbConnector.branch <stable>

git submodule update --remote <DbConnector>
git submodule update --remote
```
这时我们运行 git status，Git 会显示子模块中有 “新提交”
```
git status
```

# 移除
```
$ git clean -fdx
Removing CryptoLibrary/

$ git checkout add-crypto
Switched to branch 'add-crypto'

$ ls CryptoLibrary/
$ git submodule update --init
Submodule path 'CryptoLibrary': checked out 'b8dda6aa182ea4464f3f3264b11e0268545172af'

$ ls CryptoLibrary/
Makefile    includes    scripts        src
```

# 问题
另一个主要的告诫是许多人遇到了将子目录转换为子模块的问题。 如果你在项目中已经跟踪了一些文件，然后想要将它们移动到一个子模块中，那么请务必小心，否则 Git 会对你发脾气。 假设项目内有一些文件在子目录中，你想要将其转换为一个子模块。 如果删除子目录然后运行 submodule add，Git 会朝你大喊：
```
$ rm -Rf CryptoLibrary/
$ git submodule add https://github.com/chaconinc/CryptoLibrary
'CryptoLibrary' already exists in the index
```
你必须要先取消暂存 CryptoLibrary 目录。 然后才可以添加子模块：
```
$ git rm -r CryptoLibrary
$ git submodule add https://github.com/chaconinc/CryptoLibrary
Cloning into 'CryptoLibrary'...
remote: Counting objects: 11, done.
remote: Compressing objects: 100% (10/10), done.
remote: Total 11 (delta 0), reused 11 (delta 0)
Unpacking objects: 100% (11/11), done.
Checking connectivity... done.
```

---
**参考**
`7.11 Git 工具 - 子模块`
https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97
https://git-scm.com/book/zh-tw/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E7%B5%84-Submodules
https://git-scm.com/docs/git-submodule

`Git Submodule管理项目子模块 - nicksheng - 博客园` ★★
https://www.cnblogs.com/nicksheng/p/6201711.html

`git submodule 使用小结 - 简书`
http://www.jianshu.com/p/f8a55b972972

`使用Git Submodule管理子模块 - 姜家志 - SegmentFault`
https://segmentfault.com/a/1190000003076028