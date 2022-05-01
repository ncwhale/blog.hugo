---
title: "Transform Stickers to Misskey"
date: 2022-05-02T00:34:00+09:00
draft: false
tags: ["Misskey", "PowerShell", "ffmpeg"]
categories: ["Tool"]
isCJKLanguage: true
toc:
  enable: true
  auto: true
---

# 前言

[Misskey](https://github.com/misskey-dev/misskey) 是一个类似长毛象的去中心化微博喵~

因为建站相对容易（推荐 Docker Compose 方式），所以咱拿来作为预防 [Twitter](https://twitter.com) 被 Elon Musk 收购后可能出现审查问题的后备解决方案喵~

但是 Misskey 有少量的适配工作没做好，目前使用 WebP 格式的动画 Sticker 无法正常导入，所以作为 Walk around ， 转换成 APNG 来解决这个问题喵~

## 下载 Strickers

有很多方法了，比如你可以导出 Line 的 Stricker， 或者用 Telegram 的 Bot 帮你打包出 Stricker 包；

## 转换成 APNG

`ffmpeg.exe -vcodec libvpx-vp9 -i .\0.webp -pix_fmt rgba -vf scale=128:-1 -plays 0 0.apng`

参数说明：

1. `-vcodec libvpx-vp9`: 使用 vp9 解码，这样可以正常导出 Alpha 通道的帧；
2. `-i .\0.webp`: 指定输入文件，这里是 `0.webp`；
3. `-pix_fmt rgba`: 设置输出格式，这里是 RGBA，保证 Alpha 通道在接下来的 Filter 里保留；
4. `-vf scale=128:-1`: 设置输出尺寸，这里是 128xauto，这样可以保证输出的图片不会太大；
5. `-plays 0`: 设置播放次数，这里是 0，表示无限循环（否则 APNG 放几次就停止播放了喵）；
6. `0.apng`: 输出文件名，这里是 `0.apng`；

## PowerShell 批处理

```powershell
Get-ChildItem "." -Filter *.webp | ForEach-Object  {
  ffmpeg.exe -vcodec libvpx-vp9 -i "$($_.FullName)" -pix_fmt rgba -vf scale=128:-1 -plays 0 "$($_.BaseName).apng"
}
```