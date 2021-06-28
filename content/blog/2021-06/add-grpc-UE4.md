---
title: "Add Grpc UE4"
date: 2021-06-28T14:45:25+09:00
draft: true
---

## 前置需求

将 gRPC 链接到 UE4 里，完成一些需要三方系统转接和管理的功能；

## 方案选择

因为 UE4 本身是一个 C++ Source ， gRPC 也支持 C++ ，所以可选方案比较多：

* 直接下载 gRPC 源码并编译后作为三方库加入 UE4 项目；