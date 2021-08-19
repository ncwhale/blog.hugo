---
title: 树莓派上安装 Homeassistant 备忘
draft: false
date: 2021-08-12T17:40:15+09:00
tags: ["教程", "homeassistant", "Arch"]
categories: ["教程"]
isCJKLanguage: true
toc:
  enable: true
  auto: true
---

## 前言

这不是 Home Assistant 项目的官方教程, 但是它可以达成以下目标：

* 摆脱 Docker 的依赖（和一堆问题）
* 摆脱 Hass.IO 的控制（甚至离线运作）

## 准备阶段

* 准备好树莓派、电源、TF卡、读卡器 etc
* 下载 [Arch Linux Arm](https://archlinuxarm.org/) 的镜像(根据树莓派版本来喵)
* 根据安装提示写入镜像到 TF 卡 [以树莓派3为例](https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3)

## 安装阶段

* install [pikaur](https://aur.archlinux.org/packages/pikaur/) 或者你喜欢的 AUR 包管理器喵~
* `pikaur -Syu` 滚滚挂一下喵~
* `pikaur -S python38` 安装 Python 3.8 包（因为 HASS 还不支持 Python 3.9
* `sudo adduser homeassistant` 添加一个跑 HASS 的用户
* `sudo -u homeassistant <tmux/bash/fish...>` 切换到该用户 Shell
* `virtualenv virtualenv --python=/usr/bin/python3.8 homeassistant` 准备 venv, 并指定 Python 版本
* `source homeassistant/bin/active` 加载环境
* `pip install homeassistant -upgrade` 安装/升级 HASS ， 要注意的是，HASS 严格指定了包括 pip 在内的众多 Python 包版本，除了这个包其它包都不要手动升级，否则会被 HASS 自动降级喵……

## 配置阶段

* `hass` 先启动看看会不会报错，第一次启动它会自动pip安装大量包来进行自我补完喵……
* 浏览器开启 `http://[树莓派IP]:8123` 进行初次启动配置
* 局域网里其它机器来个 nginx 代理 SSL 和 域名绑定

## Zigbee2MQTT 集成

TBD