---
abbrlink: b481b110
categories:
- - 教程
date: '2024-06-04T18:52:56.322214+08:00'
tags:
- '2024.6'
title: 如何重置 Git 内的所有的 Commit 记录
updated: '2024-08-06T15:44:00.355+08:00'
---
> 记录一下省的我下次忘记。

- 检出到一个全新的分支：

```bash
git checkout --orphan latest_branch
```

- 在新分支中新建一个新的提交：

```bash
git add -A
git commit -am "init: init"
```

- 删除旧的主分支：

```bash
git branch -D master
```

- 重命名当前分支为主分支：

```bash
git branch -m master
```

- 强制更新远程主分支：

```bash
git push -f origin master
```
