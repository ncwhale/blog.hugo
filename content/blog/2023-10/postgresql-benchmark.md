---
title: "Postgresql Benchmark on barematel"
date: 2023-10-25T11:17:42+09:00
draft: true
tags: []
categories: []
isCJKLanguage: true
toc:
  enable: true
  auto: true
---

# 前言

在一个 PostgreSQL 讨论群组里，突然一条讨论引起了咱的兴趣：在一台物理机器的情况下，单独部署一个 PG 快，还是同时部署 citus+PG 比较快喵~

## 准备环境

基于咱已有的家用环境，咱准备单独开启一台无盘服务器来作为被测试对象，机器配置如下：

* CPU: 	Intel(R) Xeon(R) CPU E5-2670 v2 @ 2.50GHz * 2
* 内存: 128GB, 每个CPU各带 64GB
* 网络: InfiniBand QDR/Ethernet 10Gb 2P 544FLR-QSFP Adapter 工作在 IB 4*10G 模式下
* 存储：操作系统：Archiso 通过 iLO Image 载入，数据库数据：NFS（RDMA）

运行 HammerDB 的测试机则是在相同 IB 网络下的服务器，配置如下：

* CPU: 	Intel(R) Xeon(R) CPU E5-2670 v2 @ 2.50GHz * 2
* 内存: 256GB, 每个CPU各带 128GB
* 网络: InfiniBand QDR/Ethernet 10Gb 2P 544FLR-QSFP Adapter 工作在 IB 4*10G 模式下

## 测试流程

* 准备包括 citus/postgresql 安装在内的 archiso 镜像
* 使用镜像启动机器，
* 重设/分区尺寸 `mount -o remount,size=32G /run/archiso/cowspace`
* 挂载NFS分区 `mount NAS:/mnt/benchmark/ /var/lib/postgres/data `
* 使用 [pgtune](https://pgtune.leopard.in.ua/) 优化 PG 的运行时参数
* 启动 HammerDB 镜像，进行压力测试
* 每轮测试结束后，重启机器，然后以同样的镜像重新部署并开始测试

## 单独 Postgres 测试



postgresql.conf

```
# DB Version: 15
# OS Type: linux
# DB Type: mixed
# Total Memory (RAM): 64 GB
# CPUs num: 40
# Connections num: 4096
# Data Storage: san

max_connections = 4096
shared_buffers = 16GB
effective_cache_size = 48GB
maintenance_work_mem = 2GB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 300
work_mem = 512kB
huge_pages = try
min_wal_size = 1GB
max_wal_size = 4GB
max_worker_processes = 40
max_parallel_workers_per_gather = 4
max_parallel_workers = 40
max_parallel_maintenance_workers = 4
```

## 包括 4 citus 节点 测试

## 结论