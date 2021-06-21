---
title: "Linux tips"
date: 2016-05-10T08:07:05+08:00
lastmod: 2016-05-10T08:07:05+08:00
draft: true
isCJKLanguage: true
tags:
  - "入门"
---

* zsh提示命令对应软件包
```bash
source /etc/zsh_command_not_found
```

* git-p4 提交中文名文件
  
  如果在Linux下用 git-p4 提交的文件里面包含中文，设置这个变量使得 p4 不会使用 quotepath 导致提交失败。
```bash
git config --global core.quotepath false
```