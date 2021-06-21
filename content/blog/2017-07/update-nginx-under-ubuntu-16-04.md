---
title: "Update nginx under ubuntu 16.04"
date: 2017-07-15T03:00:06+08:00
lastmod: 2017-07-15T03:00:06+08:00
draft: false
isCJKLanguage: true
---

因为刚刚在 [V2EX](https://www.v2ex.com/t/375476) 看到的 nginx 爆出了中度危险漏洞，于是决定将正在用的nginx服务都升个级喵~

默认 Ubuntu 自带的 nginx 都比较 out, 正确的姿势是从官方源安装

* 在 `/etc/apt/sources.list.d/` 下添加一个 `nginx.list` 文件，内容如下：

```
deb http://nginx.org/packages/mainline/ubuntu/ xenial nginx
deb-src http://nginx.org/packages/mainline/ubuntu/ xenial nginx
```

* 添加 nginx 的 key，并更新 apt

```
curl http://nginx.org/keys/nginx_signing.key | sudo apt-key add
sudo apt update
```

* 需要注意的是，Ubuntu 自带的 nginx 系列模组会干扰nginx本体安装，所以先备份配置文件，删除ubuntu的默认模组，再重装nginx

```
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
sudo apt remove nginx nginx-common nginx-full nginx-core
sudo apt install nginx
sudo rm /etc/nginx/nginx.conf
sudo cp /etc/nginx/nginx.conf.bak /etc/nginx/nginx.conf
```

* 另外一点是此时 nginx 被 mask 了……解除并重启它：

```
sudo systemctl unmask nginx
sudo systemctl start nginx
```

* 测试无误后，加上重启自启动

```
sudo systemctl enable nginx
```