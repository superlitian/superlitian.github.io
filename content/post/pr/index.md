---
title: 📃 开源代码贡献方法

description: 通用开源库代码贡献指南，摘自xuperchain 开源文档。

slug: pr

date: 2022-10-12

categories:
- Tools

tags:
- Git
---

### 🌼 开源代码贡献指南

#### 🌿 Fork 代码
访问自己想要贡献的代码库，然后fork代码到自己的代码仓库。
![fork指南](fork.jpg)

#### 🪁 Clone 代码到本地
假设fork完之后的代码仓库路径为 🔗 <font color="orange">git@github.com:uaanaa/xuperchain.git</font>
```shell
git clone git@github.com:uaanaa/xuperchain.git
```
克隆到本地之后设置一个 **upstream** 的 **remote** 地址，方便我们同步原始仓库地址的更新
```shell
git remote add upstream https://github.com/xuperchain/xuperchain.git
```

#### ♎ 同步代码&建立分支
每次要提交PR的时候都要 **新建** 一个分支，这样可以同时开发多个feature，分支基于 **upstream** 的 **master** 建立
```shell
# 拉取上游的最新代码
git fetch upstream

# 建立新分支
git checkout -b new_feature upstream/master
```

#### 🟢 提交代码
当我们的代码写完之后就可以提交了，注意我们这里提交的remote是 origin，也就是自己的代码仓库
```shell
git push origin new_feature
Counting objects: 3, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 286 bytes | 286.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
remote:
remote: Create a pull request for 'new_feature' on GitHub by visiting:
remote:      https://github.com/uaanaa/xuperunion/pull/new/new_feature
remote:
To https://github.com/uaanaa/xuperunion.git
* [new branch]      new_feature -> new_feature
```

#### 🇨🇳 创建PR
提交完之后，一般有个类似 https://github.com/uaanaa/xuperunion/pull/new/new_feature 这样的地址，在浏览器打开这个地址就跳转到创建PR的页面进行提交即可。