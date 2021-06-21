---
title: "使用ipfilter过滤GFW滴DNS污染喵~"
date: 2012-08-27T13:46:43+08:00
lastmod: 2012-08-27T13:46:43+08:00
draft: false
isCJKLanguage: true
---

嗯嗯，最原始文档请参考AutoVPN上滴 <a href="http://code.google.com/p/autoddvpn/issues/detail?id=120" target="_blank">这篇文章</a> 喵~

咱将其原始脚本修订了一些，并添加了 OpenWRT滴正确使用姿势喵~

一、OpenWRT的准备工作

普通Linux用户可以直接跳过这一节，因为大多数发行版已经安装了这些必要的软件包了喵~
<pre lang="sh">
opkg update
opkg install iptables-mod-filter bind-dig
</pre>

二、脚本本体及注释

<pre lang='bash'>
#!/bin/ash

#这行是用一个不存在的域名解析来钓出强制跳转地址

NONEXISTDOMAIN="non.exist.domain.cn"

#这行写入一些会被污染的域名
POSIONEDDOMAIN="www.twitter.com www.facebook.com www.youtube.com plus.google.com"

#下面这行写入本地的DNS地址，即被污染的DNS服务地址
WALLSERVER="61.139.2.69"
LOOPTIMES=9

badip=""

#钓出非法域名强制跳转地址

for DOMAIN in $NONEXISTDOMAIN ; do
for IP in $(dig $DOMAIN +time=1 +tries=1 +retry=0 | grep ^$DOMAIN | grep -o -E "([0-9]+\.){3}[0-9]+") ; do
if [ -z "$(echo $badip | grep $IP)" ] ; then
badip="$badip $IP"
fi
done
done

echo First Step: $badip

#钓出污染的IP地址(GFW反馈的)

for i in $(seq $LOOPTIMES) ; do
for DOMAIN in $POSIONEDDOMAIN ; do
for IP in $(dig @$WALLSERVER $DOMAIN +time=1 +tries=1 +retry=0 | grep ^$DOMAIN | grep -o -E "([0-9]+\.){3}[0-9]+") ; do
if [ -z "$(echo $badip | grep $IP)" ] ; then
badip="$badip $IP"
echo $IP $DOMAIN
fi
done
done
done

echo Second Step: $badip

#将这些地址在 Firewall里面过滤掉
for IP in $badip
do
hexip=$(printf '%02X ' ${IP//./ }; echo)
# echo $hexip
iptables -I INPUT -p udp --sport 53 -m string --algo bm --hex-string "|$hexip|" --from 60 --to 180 -j DROP
iptables -I FORWARD -p udp --sport 53 -m string --algo bm --hex-string "|$hexip|" --from 60 --to 180 -j DROP
done
</pre>

三、原理介绍

因为DNS是使用UDP数据包进行发送的，正常的查询如下：

PC 查询域名 -查询UDP包-&gt; 网关 -查询UDP包-&gt; DNS服务器 -返回UDP包-&gt; 网关 -返回UDP包-&gt; PC结束查询

被GFW污染之后，GFW抢先在DNS服务器正确包到达之前返回被污染结果：

PC 查询域名 -查询UDP包-&gt; 网关(路由旁路触发GFW) -查询UDP包-&gt; DNS服务器 -返回UDP包-&gt; (GFW抢先返回)网关 -GFW返回UDP包-&gt; PC结束查询 -正确的UDP包-&gt; 被遗弃

但是GFW只能固定返回几个IP(返回正确IP风险很高，比如暴风上次导致的故障),所以只要找到这几个IP，加以过滤，则能够将GFW抢答的包在防火墙上设置无效丢包即可，正确的包就会在稍加等待后返回。