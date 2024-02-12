---
title: "给 Hugo 支持的静态博客添加 HTMX 支持"
date: 2024-01-10T20:42:11+09:00
draft: false
tags: []
categories: ["教程"]
isCJKLanguage: true
toc:
  enable: true
  auto: true
---

# TLNR

1. Download [htmx.min.js](https://unpkg.com/htmx.org/dist/htmx.min.js) into `static\js\` folder.
2. Append `<script async defer src="{{ "js/htmx.min.js" | relURL }}"></script>` into `layouts\_default\baseof.html` head block.
3. Append `hx-boost="true"` to `<div class="container container--outer">` in the same file.

# 太长不看教程

1. 下载 [htmx.min.js](https://unpkg.com/htmx.org/dist/htmx.min.js) 放到 `static\js\` 目录下；
2. 添加 `<script async defer src="{{ "js/htmx.min.js" | relURL }}"></script>` 代码到 `layouts\_default\baseof.html` 文件的 `<head>` 里；
3. 在同一文件种，添加 `hx-boost="true"` 标签到 `<div class="container container--outer">` 里。

## 实际用途

直接就将一个静态博客变成类似SPA的体验了喵！非常神奇喵！在博客内跳转再也不会闪烁+加载一堆CSS等操作了，而仅仅只是将内容替换掉喵！

并且，就算浏览器古代到不支持 JavaScript，整个静态站还是能正常工作的喵！

## 缺陷

目前已知问题是 Disqus 的评论区会引起链接失去HTMX的支持变成全页重载喵。