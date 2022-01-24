---
title: "Ark Linux Server Setup"
date: 2022-01-23T22:36:10+09:00
draft: false
tags: ["Linux", "Ark", "Server"]
categories: ["Service"]
isCJKLanguage: true
toc:
  enable: true
  auto: true
---

# 前言

总之就是看到各种 Hololive Ark 废人生成视频太多，于是想自己也弄个玩玩喵~

虽然直接在本地跑个服务器也问题不大但是这样就没法找人来玩了喵……

于是，Gen8 再次变成了多用途机器喵~

## 系统需求

搜索了一下，因为就是 UE 的 Dedicated server 框架，所以服务端所要资源其实不多喵~ 

CPU 2Core 以上， 8G 内存， 外带20~30G 磁盘就足够喵~

Linux / Win 系统均可（因为是直接通过 Steam 分发的喵），咱是 Arch 直接 [AUR:steamcmd](https://aur.archlinux.org/packages/steamcmd/) 就好喵~

## 开始安装

首先安装 steamcmd

    $ sudo pikaur -S steamcmd

然后为服务端创建一个文件夹


    $ mkdir -p ~/ark-server

备注：更好的做法是新建一个专门的用户并指定它的 `$HOME` 目录到 `<anywhere>/ark-server`，这样就不用担心服务端被意外访问，并可缩小该用户权限以保证安全喵。

    $ sudo adduser -d <somewhere>/ark-server  -Nml -s /usr/bin/nologin ark-server 
    $ sudo -u ark-server </bin/anyshell> # 因为我们指定该用户默认 shell 为 `nologin` 所以无法直接登入

使用 `steamcmd` 下载服务端:

    $ steamcmd +force_install_dir ~/ark-server +login anonymous +app_update 376030 validate +quit

注意:

* 如果是该用户第一次运行 `steamcmd` ，请使用无任何参数的版本运行一次后 `exit` 再执行上述指令喵~
* 要先指定 `+force_install_dir` 参数再登录 `login`，否则会报错喵~（Wiki里记载的顺序有误喵）
* 如果使用独立用户执行 `steamcmd` ，可以不用指定 `+force_install_dir` （将会安装到该用户的 `$HOME/.steam/steamapps/common/ARK Survival Evolved Dedicated Server/` 目录下）喵~
* app id 376030 => 生存进化(Survival Evolved),  445400 => 适者生存(Survival of The Fittest)e 
* `+app_update` 参数后面的 `validate` 参数是为了验证安装/升级是否成功

## 启动脚本

非常建议写个 Shell 脚本来启动服务器，测试OK之后还可以通过 `systemd` 的方式来自动启动喵~

注意启动脚本参数里包含服务器 Admin 密码， 所以请注意修改成随机密码并不要泄漏喵~

```bash
#!/bin/bash

ShooterGame/Binaries/Linux/ShooterGameServer TheIsland?listen?SessionName="Kiri no Mizuumi"?MaxPlayers=12?ServerAdminPassword=<PLACEHOLDER>?DifficultyOffset=0.200000?NewMaxStructuresInRange=6000.000000?GlobalVoiceChat=false?ProximityChat=false?NoTributeDownloads=false?AllowThirdPersonPlayer=true?AlwaysNotifyPlayerLeft=false?DontAlwaysNotifyPlayerJoined=false?ServerHardcore=false?ServerPVE=false?ServerCrosshair=true?ServerForceNoHUD=false?ShowMapPlayerLocation=false?EnablePvPGamma=true?DisableStructureDecayPvE=false?AllowFlyerCarryPvE=false -nosteamclient -game -server -log
```

备注： 如果发现服务启动不起来，可能你需要修改 `fs.max-files` 和 `ulimit` 等参数喵~

## 自动启动

这就简单了，直接用 `systemd` 就可以了喵~

因为咱还有另外一套基于 `byobu-tmux` 的自动化脚本，所以这里不再赘述喵~(相关博文还在编写，近期会发布喵~)

## 最后一步

使用下列神秘代码来加入咱的服务器喵：

```bash
sudo zerotier-cli join 0cccb752f77ab7bc
```

## 参考资料

1. [Wiki: Dedicated server setup](https://ark.fandom.com/wiki/Dedicated_server_setup#Linux)
