---
title: 🧰 git 忽略大小写问题

description: 解决git检查大小写不敏感问题

slug: git

date: 2022-10-12

categories:
- Tools

tags:
- Git
---

### 🔴 git 检查大小写不敏感
今天需要将项目一个文件名首字母从小写改为大写，改好之后发现  **git status** 并没有检查到更改。

### 🟡 原因
对于 windows 和 macOS 用户，每次 **git clone** 之后，都在该项目的 📁 **.git/config** 文件中设置了 **core.ignorecase = true**

### 🟢 解决
在终端执行以下命令，将忽略大小写改为false
```shell
git config core.ignorecase false
```
