---
abbrlink: 47021a1b
categories:
- - 教程
date: '2024-06-07T16:52:05.770777+08:00'
tags:
- '2024'
title: 如何在 Jupyter Notebook 中使用 R 语言内核
updated: '2024-08-05T10:03:10.649+08:00'
---
# 前言

我发现阿里的 Modelscope 免费给别人提供了深度学习的 Jupyter 在线笔记本服务，但是谷歌的 Colab 是提供 R 语言核心切换的，没有的话就自己搓。

# 安装

Modelscope 的在线笔记本的环境是 Ubuntu，因而先刷新包管理并更新：

```bash
sudo apt update
sudo apt upgrade
```

安装R：

```bash
sudo apt install r-base
```

特别的，如果没有拉取到这个包，则应该是服务器没有配资源，这个问题目前不存在在 Modelscope 上，建议：

```bash
sudo apt install dirmngr gnupg apt-transport-https ca-certificates software-properties-common
sudo vim /etc/apt/sources.list
```

添加内容：

```plaintext
deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/
```

更新源然后再安装：

```bash
sudo apt update
sudo apt install r-base
```

# R插件支持

在 Bash 输入```R```，之后：

```bash
install.packages('IRkernel')
IRkernel::installspec(user = FALSE)
```

刷新你的笔记本，这样应该能在笔记本右上角切换内核为 R 了。

# 二三事

经过一系列使用感觉 Modelscope 的环境还是太不稳定了，建议还是用 Colab。
