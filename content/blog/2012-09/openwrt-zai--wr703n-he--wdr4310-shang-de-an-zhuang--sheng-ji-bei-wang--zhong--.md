---
title: "OpenWRT 在 WR703n 和 WDR4310 上的安装\\升级备忘(中)"
date: 2012-09-05T11:39:35+08:00
lastmod: 2012-09-05T11:39:35+08:00
draft: false
isCJKLanguage: true
---

此前在OpenWRT Trunk 版本上一直存在的VPN掉线问题终于在12.9 Beta版本中解决了喵！撒花！ 于是咱们可以进一步刷机制作无界智能路由器了喵~

(链接：<a href="http://forum.openwrt.org/viewtopic.php?pid=176925#p176925" target="_blank">Attitude Adjustment (12.09-beta)</a>

那么，接下来就素刷机升级流程备忘和注意事项哦喵~

一、准备升级文件喵~

请下载下面两个文件到本地计算机备份一个哦喵~否则等下如果出错滴话就麻烦了呢喵~有备无患喵~

原厂固件请使用这两个文件进行刷机：
<ul>
	<li><a href="http://downloads.openwrt.org/attitude_adjustment/12.09-beta/ar71xx/generic/openwrt-ar71xx-generic-tl-wr703n-v1-squashfs-factory.bin" target="_blank">WR-703n</a></li>
	<li><a href="http://downloads.openwrt.org/attitude_adjustment/12.09-beta/ar71xx/generic/openwrt-ar71xx-generic-tl-wdr4310-v1-squashfs-factory.bin" target="_blank">WDR-4310</a></li>
</ul>
已经刷入OpenWRT其它版本请使用这两个链接喵：
<ul>
	<li><a href="http://downloads.openwrt.org/attitude_adjustment/12.09-beta/ar71xx/generic/openwrt-ar71xx-generic-tl-wr703n-v1-squashfs-sysupgrade.bin" target="_blank">WR-703n</a></li>
	<li><a href="http://downloads.openwrt.org/attitude_adjustment/12.09-beta/ar71xx/generic/openwrt-ar71xx-generic-tl-wdr4310-v1-squashfs-sysupgrade.bin" target="_blank">WDR-4310</a></li>
</ul>

二、开始刷机喵~

在刷机之前，如果之前已经做了N多配置等东西，建议将其备份到外置U盘上哦喵~因为这次刷机需要完整清理之前的文件，否则可能导致各种无法Boot滴故障哦喵~比如这样：

[cc lang="bash"]
cp -rf ../etc/openvpn /overlay/root
cp -rf ../etc/ppp /overlay/root
cp -rf ../etc/config /overlay/root
[/cc]

接下来使用 [cci lang="bash"]top[/cci]查看是否有足够的内存进行升级，起码要有Free 4MB内存哦喵~

[cc lang="bash"]
cd /tmp/
wget <a href="http://downloads.openwrt.org/attitude_adjustment/12.09-beta/ar71xx/generic/openwrt-ar71xx-generic-tl-wdr4310-v1-squashfs-sysupgrade.bin">http://downloads.openwrt.org/attitude_adjustment/12.09-beta/ar71xx/generic/openwrt-ar71xx-generic-tl-wdr4310-v1-squashfs-sysupgrade.bin</a>
#请一定记得加 –n 参数，否则会无限重置/无法启动等问题喵。
sysupgrade -n openwrt-ar71xx-generic-tl-wdr4310-v1-squashfs-sysupgrade.bin
[/cc]

如果乃和咱一样爪子摁错了参数，导致路由器启动不了了，请断电->找一只细长的东西捅住 Reset ->上电->等10秒钟-> telnet 192.168.1.1

然后，使用 cd /tmp/ && wget && sysypgrade -n 再来一次喵。此时前面备份滴升级文件就有用啦喵~

三、恢复配置等

接下来等待路由器自动重置，然后按照 上一篇 介绍那样重新设置路由器到能够正确工作哦喵~
如果在步骤二开始备份了配置的话，这里就相对简单一点了呢喵~直接恢复回来就好喵~
那么，将有线LAN插到路由器上，进行下面动作吧喵~

[cc lang="bash"]
telnet 192.168.1.1
passwd
******
******
exit
[/cc]

对了，新的发布版本自带了Luci，所以下面的配置都可以通过 <a href="http://192.168.1.1">http://192.168.1.1</a> 来进行，当然，咱喜欢命令行喵~更简单快捷喵~

[cc lang="bash"]
ssh root@192.168.1.1
vi /etc/config/network
vi /etc/config/wireless
/etc/init.d/network restart
#注意，这里不要一开始就安装 block-mount 因为还需要清理既有U盘的陈旧文件和配置
opkg update && opkg install kmod-fs-ext4 kmod-usb-stroage
cd /tmp
mkdir usb
mount /dev/sda1 /tmp/usb
cd usb
#除了 /home 和 /root 其它全部删除喵~否则会~各种出错哦喵~
rm -rf etc/ lib/ mnt/ sbin/ usr/ www/
opkg install block-mount
vi /etc/config/fstab
#然后将新的 /overlay 拷贝回来
cp -rf /overlay/* /mnt/usb/
reboot
[/cc]

等启动完毕，U盘再次被正确挂载后，使用 cp 来恢复已经备份的配置吧喵~
[cc lang="bash"]
opkg update && opkg install nano #名乃编辑器！小文档最爱喵！
cp /root/... /... #根据自己需要复制啦喵~
reboot #/etc/init.d/ 相关服务重新启动也可以喵~
[/cc]

四、设置OpenVPN进行自动爬墙喵~

首先说一下，咱的配置是针对 3DS、PSV 两个设备无论如何都走日本VPN出口，对他们设置源路由
剩下的依照河蟹网站和部分地区等设置目标路由
对国内的所有地址设置目标路由(大概2600条)
由于设置了路由分表，所以其实路由速度还是飞快的喵~(用空间换取时间

[cc lang="bash"]
opkg install openvpn ip #用它们两个组件就OK了喵~
opkg install luci-app-openvpn #纯备选，因为咱都用nano搞定一切配置所以装不装都OK喵。
nano /etc/config/openvpn #UCI配置文件，对于OpenVPN如果想要做复杂配置还是自己编辑.cfg好喵~
nano /etc/openvpn/vpn.cfg #咱的OpenVPN配置文件，里面有各种调用
nano /etc/openvpn/vpnup.sh #自动翻墙脚本喵~重点在这里哦喵~
chmod a+x /etc/openvpn/vpnup.sh
[/cc]

/etc/config/openvpn样例，只需要几句话就好哦喵~
[cc]
package openvpn

config openvpn my_vpn_config
        option enabled 1
        option config /etc/openvpn/vpn.cfg
[/cc]

/etc/openvpn/vpn.cfg 样例，注意 route-up 和 script-security 2 两行，其它的请根据自己的服务器配置喵~
[cc]
#服务器有多线的情形下，自己找较好的几条配置在下面，然后随机连接
remote-random
remote jp.xxx.net 443
remote hk.xxx.net 21
remote us.xxx.net 99
remote ca.xxx.net 1008

client
dev tun

#请根据服务器和自身网络设置喵~
#tun-mtu 1300
#mssfix 1300
#link-mtu 1420
proto udp #UDP效率高喵~
resolv-retry infinite #无限重连，路由器必备
nobind
auth-user-pass /etc/openvpn/xxx.txt
#auth-nocache #这句不注释掉，自动重连可能出错

#这些都看自己服务器的配置哦喵~
ca /etc/openvpn/ca.crt
persist-key
persist-tun
ns-cert-type server
comp-lzo

#路由设置，这里是重点喵~自动翻墙就靠这些了喵~
route-delay 2
#route-method exe
#route-nopull

#通过服务器获得网关，但是不自动配置，使用脚本喵~
route-gateway 'dhcp'
route-noexec
route-up /etc/openvpn/vpnup.sh
#这个对于长期连接的路由来说几乎没用喵~当然也可以写上方便调试喵~
down /etc/openvpn/vpndown.sh
#这个必须设置，才能正确运作脚本喵~
script-security 2
verb 3
[/cc]

/etc/openvpn/vpnup.sh 样例，请根据自动的网络需求修订哦喵~
(备注: 这个脚本可以进一步完善，比如连接上 被墙网站列表 自动更新列表后，再反转查询所有域名对应IP最后自动设置路由喵~

[cc lang="bash"]
#!/bin/ash
#设置路由表⑨，注意这里设置的是 VPN 网关在前，如果VPN断掉，那么会继续走默认网关喵~
#所以没必要写 VPN Down 的脚本，这里会自动搞定的喵。
ip route flush table 9
ip route add default table 9 dev $dev via $route_vpn_gateway metric 10
ip route add default table 9 dev pppoe-wan via $route_net_gateway metric 20
ip route add 192.168.1.0/24 table 9 dev br-lan

#注意，如果不需要动态设置这些IP，那么完全可以放在 rc.local 里面喵~
#同时记得在 /etc/ppp/ipup.d/里面写一个 route9 的表，使得拨号成功后
#所有设备都能上网，只是少量撞墙而已喵~

#设置源路由，从这里来的数据都走⑨表出去喵~
ip rule del pref 200
ip rule add from 192.168.1.9 pref 200 table 9

#设置目标路由，所有访问这个IP/IP段的数据都从⑨表出去喵~
ip rule del pref 9
ip rule add to 8.8.8.8 pref 9 table 9

#这里可以设置China的全部走默认路由，不过，一般……都不必设置喵~

#这里的日志可以不要，咱纯用来自己观察的喵~
echo `date` > /tmp/vpn.log
#echo dev: $dev >> /tmp/vpn.log
echo ifconfig_local: $ifconfig_local >> /tmp/vpn.log
echo route_net: $route_net_gateway >> /tmp/vpn.log
echo route_vpn: $route_vpn_gateway >> /tmp/vpn.log
echo ==============ENV============== >> /tmp/vpn.log
env >> /tmp/vpn.log

exit 0
[/cc]

五、设置DNS解析喵~
重点：国内常用网站及DDNS加速站点走默认DNS，其余站点全部走OpenDNS或者GoogleDNS喵~

[cci]nano /etc/config/dhcp[/cci]
[cc]
config dnsmasq
        option domainneeded '1'
        option boguspriv '1'
        option localise_queries '1'
        option rebind_protection '1'
        option rebind_localhost '1'
        option local '/lan/'
        option domain 'lan'
        option expandhosts '1'
        option authoritative '1'
        option readethers '1'
        option leasefile '/tmp/dhcp.leases'
        option resolvfile '/tmp/resolv.conf.auto'
		#下面就设置各个server 网上有更全的设置脚本喵~
        list server '/cn/61.139.2.69'
        list server '/jp/8.8.8.8'
        list server '/us/8.8.8.8'
        list server '/net/8.8.8.8'
        list server '/com/8.8.8.8'
        list server '/iask.com/61.139.2.69'
        list server '/qq.com/61.139.2.69'
        list server '/163.com/61.139.2.69'
        list server '/baidu.com/61.139.2.69'
        list server '//8.8.8.8'

config dhcp 'lan'
        option interface 'lan'
        option start '100'
        option leasetime '12h'
        option limit '250'
        option force '1'
        list dhcp_option '6,192.168.1.1'

config dhcp 'wan'
        option interface 'wan'
        option ignore '1'

config host
        option name 'PSV'
        option mac 'd4:4b:5e:24:78:e6'
        option ip '192.168.1.9'

config host
        option name 'Nexus7'
        option mac '10:bf:48:f5:e9:e1'
        option ip '192.168.1.201'

[/cc]
