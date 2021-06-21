---
title: "Rabbitmq on Wsl2"
date: 2021-06-21T15:51:08+09:00
tags: ["rabbitmq", "WSL", "教程"]
categories: ["教程"]
toc: true
---

## 首先，把兔子抓过来

基于 RabbitMQ 官方的[安装步骤](https://www.rabbitmq.com/install-debian.html#apt-quick-start-cloudsmith) 使用了 `Cloudsmith Quick Start Script` 在 `Ubuntu 20.04 @ WSL2` 里进行安装：

```sh
#!/bin/sh

sudo apt-get install curl gnupg debian-keyring debian-archive-keyring apt-transport-https -y

## Team RabbitMQ's main signing key
sudo apt-key adv --keyserver "hkps://keys.openpgp.org" --recv-keys "0x0A9AF2115F4687BD29803A206B73A36E6026DFCA"
## Cloudsmith: modern Erlang repository
curl -1sLf https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/gpg.E495BB49CC4BBE5B.key | sudo apt-key add -
## Cloudsmith: RabbitMQ repository
curl -1sLf https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/gpg.9F4587F226208342.key | sudo apt-key add -

## Add apt repositories maintained by Team RabbitMQ
sudo tee /etc/apt/sources.list.d/rabbitmq.list <<EOF
## Provides modern Erlang/OTP releases
##
deb https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/deb/ubuntu bionic main
deb-src https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-erlang/deb/ubuntu bionic main

## Provides RabbitMQ
##
deb https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/deb/ubuntu bionic main
deb-src https://dl.cloudsmith.io/public/rabbitmq/rabbitmq-server/deb/ubuntu bionic main
EOF

## Update package indices
sudo apt-get update -y

## Install Erlang packages
sudo apt-get install -y erlang-base \
                        erlang-asn1 erlang-crypto erlang-eldap erlang-ftp erlang-inets \
                        erlang-mnesia erlang-os-mon erlang-parsetools erlang-public-key \
                        erlang-runtime-tools erlang-snmp erlang-ssl \
                        erlang-syntax-tools erlang-tftp erlang-tools erlang-xmerl

## Install rabbitmq-server and its dependencies
sudo apt-get install rabbitmq-server -y --fix-missing
```

如果一运行 `sudo rabbitmqctl` 就出现如下提示，需要再安装 `libncurses5` 喵

    /usr/lib/erlang/erts-12.0.2/bin/beam.smp: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory

```sh
## 增补：WSL2 版本的 Ubuntu 没有安装 libncurses5 喵……
sudo apt install libncurses5
```

## 其次，让兔子打洞

因为 WSL2 下的 Ubuntu 默认没有 Systemd 子系统在跑（看top就只有个 init 0) ， 所以启动兔子服务需要手动来喵……

```sh
# 开发阶段推荐 TMUX 一类分终端跑喵~正式部署还是 Systemd 系统喵~
sudo -u rabbitmq rabbitmq-server
```

## 然后，跳进兔子洞

~~也变不成爱丽丝喵~~

注意的是，WSL2 的网络层非常坑喵~所以教程里的 `localhost` 即使在同一台 Windows10 下也是没用的喵……你得用 WSL2 分配给虚拟机的 IP 地址喵……