---
title: "通过自己编写PAC文件实现飞速土豆共享~"
date: 2008-09-03T14:54:37+08:00
lastmod: 2008-09-03T14:54:37+08:00
draft: false
isCJKLanguage: true
---

首先BS一下公司的网络设置，无与伦比的鄙视，有线网络无法直接访问互联网，只能通过无线网卡，而无线网卡又限定了只有那几个可以使用= =!于是，台式机要上网嘛……CCProxy+SocksCap~

可是呢，比如飞速土豆等软件似乎对SocksCap消化不良，无法正确飞起来，始终是有连接却无数据- -!

于是愤怒的我拆了它的自动生成的PAC文件来看：<br />
<pre><code class="html">
function FindProxyForURL(url, host) {
if(isPlainHostName(host) || url.substring(0,5) != "http:" )
  return "DIRECT";
if(shExpMatch(url, "*.flv*") || shExpMatch(url, "*.mp4*"))
 {  if(shExpMatch(url, "*hzplayer0.tudou.com*")) return "DIRECT";
      else return "PROXY localhost:9415"; }
else return "DIRECT"; }</code></pre>
原来真相这么容易大白于天下啊~这个文件就是一个脚本，比较HTTP请求后就给出具体的代理流向，而<em>localhost:9415</em>就是偶们的飞速土豆开的代理端口。
于是写出如下PAC文件来让局域网上其它浏览器也能飞起来~顺道还将使用http代理配置。注意下面代码中，192.168.0.12是偶的局域网开了飞速土豆和代理的地址。请替换成你自己的服务器的地址。<br />
<pre><code class="html">function FindProxyForURL(url, host)
 { if(isPlainHostName(host) || url.substring(0,5) != "http:" )
      return "PROXY 192.168.0.12:808";
  if(shExpMatch(url, "*.flv*") || shExpMatch(url, "*.mp4*"))
  { if(shExpMatch(url, "*hzplayer0.tudou.com*")) return "PROXY 192.168.0.12:808";
     else return "PROXY 192.168.0.12:9415"; }
 else return "PROXY 192.168.0.12:808"; }</code></pre>