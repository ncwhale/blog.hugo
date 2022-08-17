---
title: "Zfs Tips"
date: 2022-08-11T12:34:52+09:00
draft: false
tags: ["Tool", "Samba", "ZFS", "NFS"]
categories: ["Tool"]
isCJKLanguage: true
toc:
  enable: true
  auto: true
---

# ZFS Tips for Home NAS

## Automatic Mount when Boot

事实上，ZFS 现在已经通过设置多个 systemd .service 和 .target 来开启自动挂载功能。[^1]

```bash

sudo systemctl enable zfs-import-cache.service
sudo systemctl disable zfs-import-scan.service
sudo systemctl enable zfs-mount.service
sudo systemctl enable zfs-share.service
sudo systemctl enable zfs-zed.service
sudo systemctl enable zfs.target
```

[^1]: [ZOL version 0.6.5.8 - WARNING](https://github.com/archzfs/archzfs/issues/72)

## Automatic Share with Samba using sharesmb property

1. 安装并配置 Samba ， 关键是要在 `/etc/samba/smb.conf` 里配置好 `[global]` 部分[^2]：

```ini
[global]
...
   usershare path = /var/lib/samba/usershares
   usershare max shares = 100
   usershare allow guests = yes
   usershare owner only = no
```

2. 建立 `/var/lib/samba/usershares` 目录，并设置权限：

```bash
sudo mkdir -p /var/lib/samba/usershares
sudo chmod +t /var/lib/samba/usershares
```

3. 启动或者重启 Samba 服务：

```bash
sudo systemctl restart smbd.service
```

4. 自动设置 ZFS dataset 分享：

```bash
sudo zfs set sharesmb=on /tank/volume
```

[^2]: [Newbie Corner » [SOLVED] ZFS sharesmb](https://bbs.archlinux.org/viewtopic.php?id=185884)