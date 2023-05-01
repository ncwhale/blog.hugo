---
title: "Revanced Step"
date: 2022-10-21T23:40:52+09:00
draft: false
tags: ["Tool", "revanced", "youtube", "twitter", "spotify"]
categories: ["Tool", "教程"]
isCJKLanguage: true
toc:
  enable: true
  auto: true
---

# ReVanced 去掉APP里烦人的广告喵！

## 前言

一年前，有一个 `Vanced YouTube` 项目，通过修改 U2B 的 APK 包来免除广告，得到和 YouTube Premium 一样的体验——然后这个项目就被律师函了，现在项目已经关闭，只有两个付费广告拦截插件了喵。

然后，开源社区又有人举起了旗帜，开发了 [ReVanced](https://github.com/revanced) 套件，该套件自身并不分发 APK 包，仅提供对APK打补丁的工具和对应的补丁，绕开了 APK 的版权问题喵~

本文基本上是 [ReVanced Documentation](https://github.com/revanced/revanced-documentation) 的中文翻译，增加了一些需要的额外说明和补充而已喵~如果和原本 wiki 内容有出入则以原本 wiki 为准喵~

为了减轻大家工作量，这里会直接使用预编译的包————如果想要手动编译一切，请参考 [Building from source](https://github.com/revanced/revanced-documentation/wiki/Building-from-source) 节喵~

## 准备工作

1. 如果是 Windows PowerShell 里推荐安装 [scoop](https://scoop.sh/) ([教程](/blog/2021-06/posershell-tips/)) 作为包管理器，接下来的步骤也会用 `scoop` 来示范安装需要的包；Linux 系统请使用各个发行版自带的包管理；

2. 安装 `zulu17-jdk` 包，这个包是 `ReVanced` 用来运行和重新打包 APK 需要的 Java 运行环境：

```POWERSHELL
scoop bucket add java # 添加 Java bucket
scoop install zulu17-jdk adb # adb 用于自动装包
```

3. 新建一个目录，下载这些预编译包：

    注意： 在本文撰写完成后，可能版本会有变化，最好去对应 Release 页面下载最新版本。

    [Micro G](https://microg.org/) 是 YouTube / YouTube Music 在 ReVanced Patch 后需要的无广告轻量 Google 平台（很多时候闪退就是因为没装这个喵）

```POWERSHELL
# https://github.com/revanced/revanced-cli/releases/
wget "https://github.com/revanced/revanced-cli/releases/download/v2.20.2-dev.1/revanced-cli-2.20.2-dev.1-all.jar" 

# https://github.com/revanced/revanced-patches/releases
wget "https://github.com/revanced/revanced-patches/releases/download/v2.172.0-dev.9/patches.json"
wget "https://github.com/revanced/revanced-patches/releases/download/v2.172.0-dev.9/revanced-patches-2.172.0-dev.9.jar"

# https://github.com/revanced/revanced-integrations/releases
wget "https://github.com/revanced/revanced-integrations/releases/download/v0.106.0-dev.7/revanced-integrations-0.106.0-dev.7.apk"

# Micro G https://github.com/microg/GmsCore/releases
wget "https://github.com/microg/GmsCore/releases/download/v0.2.27.223616/com.google.android.gms-223616054.apk"
wget "https://github.com/microg/GmsCore/releases/download/v0.2.27.223616/com.google.android.gms-223616054.apk.asc"
```

## 开始打补丁喵~

首先在这里确认你想要打补丁的 apk 包版本喵： [Patches](https://github.com/revanced/revanced-patches)

这里我们以 Twitter 为例，去掉 TL 上的广告并做一些个性化修订喵：

1. 确认 `com.twitter.android` 补丁适用的 `Target Version`；
2. 前往 [APKMirror](https://www.apkmirror.com/) 上搜索包名字并选择对应的 `APK` 包下载后放入刚刚下载环境的相同目录里；

    注意： 不要下载 bundle 包而要下载完整 APK 包，否则可能打补丁失败（特别是要修改 Res 的情况）

3. 确认刚刚下载的 `patches.json` 里有你选择的补丁（并且可以调整参数）；
4. 打开爪机的 USB/Wifi 调试模式，并连接到电脑，授权调试；
5. 使用 `adb devices` 确认设备的ID；
6. 使用 `java -jar .\revanced-cli-2.14.0-all.jar -a "com.twitter.android_9.63.0-release.0-29630000_minAPI21(arm64-v8a,armeabi-v7a,x86,x86_64)(nodpi)_apkmirror.com.apk" -c -d DEVICE_ID_HERE -o "com.twitter.android_9.63.0-release.0-29630000_minAPI21(arm64-v8a,armeabi-v7a,x86,x86_64)(nodpi)_revanced.apk" -b revanced-patches-2.82.1.jar` 来将刚刚下载的 APK 包打补丁并安装到爪机里

注意：

*  如果被补丁的是 Google 相关的包（Youtube/Music etc），则还需要 `adb install .\microg.apk` 把 MicroG 安装进去，否则会闪退；
*  某些APP 需要 `revanced-integrations` 的支持才能修补，此时需要再步骤6那一长串指令后补充 `-m <revanced-integrations.apk>` 喵~

## 完成！

其实，这些操作对于普通人来说还是过于繁杂了，所以 `ReVanced` 项目有推出一个 [ReVanced Manager](https://github.com/revanced/revanced-manager) 来让大家在爪机上完成所有步骤，但是现在还在 Alpha 阶段，只能说 It works 但是并不稳定喵……等一个社区后续喵~

## CHANGELOG

* 2023-05-01: 更新了各个包的链接版本；
* 2023-05-01: 增加了一些注意事项喵~