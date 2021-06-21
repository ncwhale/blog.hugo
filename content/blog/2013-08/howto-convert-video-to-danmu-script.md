---
title: "Howto:将BA转换成弹幕视频"
date: 2013-08-21T15:46:17+08:00
lastmod: 2013-08-21T15:46:17+08:00
draft: false
isCJKLanguage: true
tags:
  - "坑"
---

所需工具

mplayer/mencoder/flac/x264 ——用来剪切视频

Inkscape/potrace 用来进行矢量图转换(SVG)

基本流程

<span style="line-height: 1.7;">1.制作黑屏视频</span>

首先切分音频流：

<code>mplayer -ao pcm -vo null Bad\ Apple\!\!\(1440X1080\)\(x264+flac\)sub.mkv </code>

将音频流和静态黑屏组合成视频流:

备注：
FPS设置成 1/220 是因为 BA 一共长 3:39.1 = 219.1s
audiodump.wav 是上面切分音轨生成的默认.wav文件

<code>mencoder "mf://white.png" -mf fps=1/220 -audiofile audiodump.wav -oac lavc -lavcopts acodec=flac -ovc x264 -ofps 10 -o ba.avi</code>

2.制作矢量图像流
for f in *.ppm; do potrace -s --group -i $f; done
3.弹幕合成本

---

这是 2013 年的脑洞了喵～并且成品已经完成，但是效率不高的PC播放会各种跳帧掉帧（虽然脚本对于这类做了很好的同步处理）——效果太渣所以一直在B站黑历史投稿里面没发表喵……