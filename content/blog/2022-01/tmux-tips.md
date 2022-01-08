---
title: "Byobu Tmux Tips"
date: 2022-01-08T17:43:20+09:00
draft: true
tags: ["Linux", "Shell", "Tmux"]
categories: ["Tool"]
isCJKLanguage: true
toc:
  enable: true
  auto: true
---

# Byobu Tmux Tips

## 前言

用 byobu 作为默认 SSH Shell 管理器有点时间了喵~ 之前都只是作为一个有方便快捷键绑定的前端，但是读文档的时候总觉得少了很多脚本自动化的东西，比如 session 管理等——直到最近才恍然大悟：要自动化这些，要看 byobu 后端的文档，而不能用 byobu 前端的脚本喵~ （自己也不知道为什么，但是觉得这样比较好，毕竟 byobu 后端的文档比较详细） <--- 这后半截都是 Github Copilot 自动生成的，咱只打了个反括号喵！

所以本文是稍微总结一下最近理顺了的 `byobu-tmux` 的使用方法，以及一些基本的技巧喵！

## 理解 tmux 是什么喵~

推荐看这篇文章来深入理解tmux：[The Tao of tmux](https://leanpub.com/the-tao-of-tmux/read)

![The Tao of tmux](https://d2sofvawe08yqg.cloudfront.net/the-tao-of-tmux/s_hero?1620552655)

简单来讲， tmux 是一个工作在 `本地` 的终端窗口管理器，它可以启动多个 服务 (Server) 每个服务下可以挂多个 会话 (session) 每个会话又可以有多个 窗口 (window) 每个窗口又可以有多个 窗格 (pane)，所有和 tmux 的交互都可以通过 tmux 指令 或者 快捷键完成喵~ 

## 理解 byobu 究竟做了什么

Byobu 其实是一个 Script wrapper，它提供了一系列预设的配置文件和工具，所以实际要自动化 tmux 的时候看 byobu的文档是不行的喵~ 此时需要用 byobu 的 `命令穿透` 方式来使用 tmux 的命令完成自动化喵！

byoby 帮我们做了以下快捷键绑定到 tmux （节选，完整版本请参考[官方文档](https://help.ubuntu.com/community/Byobu)）：

* F2 新建 window
* F3/F4 切换 window
* F6 断开当前 session 连接
* F7 进入翻页模式
* F9 启动 byobu 配置窗口

此外， Byobu 还带来了一些额外的状态栏插件，比如网络IO 磁盘IO CPU 监控等喵~

![byobu status bar](https://help.ubuntu.com/community/Byobu?action=AttachFile&do=get&target=bottom.png)

# 言归正传，开始自动化 tmux 喵~

咱的需求其实很简单，在机器启动之后，手动登录之前，自动化启动一个无头 byobu 终端并在里面执行一些命令喵~

这样咱就可以不用每次上电就手动执行一些需要 tui 界面的命令了喵~

