---
title: "Things after install ubuntu 14.04"
date: 2014-09-02T16:18:01+08:00
lastmod: 2014-09-02T16:18:01+08:00
draft: false
isCJKLanguage: true
tags:
  - "入门"
---

备忘用喵～因为最近装了太多Ubuntu 14.04 手略发麻喵……

#基本支持包
```BASH
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install language-pack-zh-hans
```

#Node.js
~~ * 使用PPA源安装Node js
```BASH
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
```
~~

* 使用 [NVM](https://github.com/creationix/nvm)
```BASH
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.0/install.sh | bash
```

#Ruby
* 安装 [RVM](https://rvm.io/rvm/install)
* 配置 Puma 自动启动脚本

#PostgreSQL
```BASH
echo 'deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main' | sudo tee /etc/apt/sources.list.d/pg.list
sudo apt-get update
sudo apt-get install postgresql libpq-dev
```

#Redis
```BASH
sudo add-apt-repository ppa:chris-lea/redis-server
sudo apt-get update
sudo apt-get install redis-server
```

#MongoDB
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/mongodb.list
sudo apt-get update
sudo apt-get install mongodb-org
```
