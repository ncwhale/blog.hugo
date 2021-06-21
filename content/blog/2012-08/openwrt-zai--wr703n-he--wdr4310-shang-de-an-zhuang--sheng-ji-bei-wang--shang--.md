---
title: "OpenWRT 在 WR703n 和 WDR4310 上的安装\\升级备忘(上)"
date: 2012-08-29T03:19:36+08:00
lastmod: 2012-08-29T03:19:36+08:00
draft: false
isCJKLanguage: true
---

(备注: < --所有这个格式的文本都是备注喵~
(备注: 博主偷懒中……于是本文只是上半节而已喵~

零、准备工作
<ol>
	<li><a href="http://www.taobao.com">购买路由器</a>(喂~</li>
	<li>在 <a href="http://openwrt.org">Openwrt.org</a> 上查找 <a href="http://wiki.openwrt.org/toh/start">支持设备</a> 对应的文档： <a href="http://wiki.openwrt.org/toh/tp-link/tl-wr703n">WR703n</a><a href="http://wiki.openwrt.org/toh/tp-link/tl-wdr4310">WDR4310</a><a href="http://wiki.openwrt.org/toh/tp-link/tl-wdr4300">WDR4300</a>

(PS: 4310 和 4300 只是国内国外版本不同而已，大部分硬件都相同喵~</li>
	<li>准备一个可持续供电的设备，比如UPS，因为升级固件时电压不稳或者停电基本就只有变砖后果。
(PS: 不推荐使用笔记本给WR703N供电，特别在没有外接AC的时候喵~</li>
	<li>准备一个8G以内的U盘，因为703N只有4MBFlash，如果要同时开LUCI加VPN加智能规则就可能导致存储空间不足。使用各种工具分区并格式化成EXT4+SWAP双分区。
(PS: 但是8G真的很浪费喵……</li>
	<li>一切确认无误进入刷机流程</li>
</ol>
一、刷入OpenWRT的ROM

刚刚买回来的路由器都是TP的ROM，此时之需要下载 factory.bin 结尾的ROM，通过路由器升级界面直接刷入OpenWRT即可。
(PS直接打开<a href="http://downloads.openwrt.org/snapshots/trunk/ar71xx/">这个页面</a>按Ctrl+F也素不错滴选择喵~
<ul>
	<li><a href="http://downloads.openwrt.org/snapshots/trunk/ar71xx/openwrt-ar71xx-generic-tl-wr703n-v1-squashfs-factory.bin">WR703N工厂ROM下载</a></li>
	<li><a href="http://downloads.openwrt.org/snapshots/trunk/ar71xx/openwrt-ar71xx-generic-tl-wdr4310-v1-squashfs-factory.bin">WDR4310工厂ROM下载</a></li>
</ul>
如果是需要升级(Trunk版本的ROM经常更新),则下载sysupdate 然后通过LUCI或者 [cci  lang="bash"]cd /tmp/ && wget (rom) && sysupgrade -v (rom) [/cci]刷新即可。
<ul>
	<li><a href="http://downloads.openwrt.org/snapshots/trunk/ar71xx/openwrt-ar71xx-generic-tl-wr703n-v1-squashfs-sysupgrade.bin">WR703N更新ROM下载</a></li>
	<li><a href="http://downloads.openwrt.org/snapshots/trunk/ar71xx/openwrt-ar71xx-generic-tl-wdr4310-v1-squashfs-sysupgrade.bin">WDR4310更新ROM下载</a></li>
</ul>
二、初始化配置，链接上网络并加载U盘为根文件系统

OpenWRT将配置都通过 uci 进行管理，所有的配置文件都在 /etc/config/ 下面，说明请参考openwrt 的 wiki:
<ul>
	<li><a href="http://wiki.openwrt.org/zh-cn/doc/uci/network">network 配置参考</a>(中文)</li>
	<li><a href="http://wiki.openwrt.org/zh-cn/doc/uci/wireless">wireless 配置参考</a>(中文)</li>
</ul>
实际操作步骤：
<ol>

	<li>链接上路由器的有线网络接口；
(因为默认Wifi不会开放出来；</li>
	<li>设置网卡处于 192.168.1.0/24 网段，因为默认OpenWRT地址是 192.168.1.1；</li>
	<li>第一次接触并初始化root密码
<code lang="bash">telnet 192.168.1.1
passwd #使用 passwd 设置初始化root密码；</code>
(重要：如果没设置，则任何人都有机会进入乃的路由器进行管理；
(提示: 如果提示找不到telnet.exe,请去控制面板-&gt;添加删除程序-&gt;添加删除Windows功能-&gt;Telnet 客户端 -&gt;确定
(提示：如果是升级上来的系统，也许telnet会默认关闭，试试看跳过这个步骤？</li>
	<li>[cci lang="bash"]ssh root@192.168.1.1[/cci]此时才开始正式配置系统,之后所有指令如无特别说明均在SSH中进行。</li>
	<li>[cci lang="bash"]cd /etc/config[/cci] 所有CUI配置都在这里</li>
	<li>/etc/config/network 示例
(备注: 因为咱的网络结构是：已经存在一个AP在工作中，所以新配置路由器都设置为通过Wifi连接到Internet，你也可以根据需要自行配置，这只是一个示例而已。
[cci lang="bash"]vi network[/cci]</li>
<code>
config interface 'loopback'
	option ifname 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config interface 'lan'
	option ifname 'eth0'
	option type 'bridge'
	option proto 'static'
	option ipaddr '192.168.1.1'
	option netmask '255.255.255.0'

config interface 'wifi'
	option proto 'static'
	option ipaddr '192.168.10.199'
	option netmask '255.255.255.0'
	option gateway '192.168.10.1'
	option dns '8.8.8.8'
</code>
	<li>[cci lang="bash"]vi wireless[/cci]</li>
<code>
config wifi-device radio0
	option type mac80211
	option channel 10
	# option macaddr 38:83:45:3f:51:c8 #因为下面会对MAC地址修正，所以这里不能用MAC来表示设备，改用下面的物理地址描述符
	option phy phy0
	option hwmode 11ng
	option htmode HT20
	list ht_capab SHORT-GI-20
	list ht_capab SHORT-GI-40
	list ht_capab RX-STBC1
	list ht_capab DSSS_CCK-40
	# REMOVE THIS LINE TO ENABLE WIFI:
	#下面这行disable直接删除就好，编辑模式d一下。

config wifi-iface
	option device radio0
	option network lan
	option mode ap
	option ssid OpenWrt
	option encryption none

config wifi-iface
	option device radio0
	option macaddr 7C:E9:D3:F3:AF:CF #MAC地址修订
	option network wifi
	option mode sta
	option ssid 'ssid you got.'
	option encryption 'wep'
	option key 'the key you got.'
</code>
	<li>配置完成后，使用下面语句让配置立即生效：
[cc lang="bash"]uci commit #检查有无错误输出，有的话修订好再继续
/etc/init.d/network restart[/cc]</li>
(备注: 现在检查一下路由器是否能连接上互联网了，接下来的所有操作都要通过互联网。

	<li>使用opkg 来获取所有必要的软件包
(唔喵: 下面的代码只有一行，恩恩，这就足够了喵~</li>
<code lang="bash">
opkg update &amp;&amp; opkg install block-mount kmod-fs-ext4 kmod-usb-storage
</code>
	<li>一切顺利的话，现在插上U盘，输入 [cci lang="bash"]dmesg[/cci]应该能看到类似这样的输出：</li>
<code lang="bash">
[ 3482.920000] SCSI subsystem initialized
[ 3483.000000] Initializing USB Mass Storage driver...
[ 3483.010000] scsi0 : usb-storage 1-1:1.0
[ 3483.010000] usbcore: registered new interface driver usb-storage
[ 3483.020000] USB Mass Storage support registered.
[ 3484.010000] scsi 0:0:0:0: Direct-Access     SanDisk  Cruzer Fit       1.26 PQ: 0 ANSI: 5
[ 3484.020000] sd 0:0:0:0: [sda] 15633408 512-byte logical blocks: (8.00 GB/7.45 GiB)
[ 3484.030000] sd 0:0:0:0: [sda] Write Protect is off
[ 3484.030000] sd 0:0:0:0: [sda] Mode Sense: 43 00 00 00
[ 3484.040000] sd 0:0:0:0: [sda] Write cache: disabled, read cache: enabled, doesn t support DPO or FUA'
[ 3484.050000]  sda: sda1 sda2 #注意这里，分区正确的话这里就能识别了
[ 3484.070000] sd 0:0:0:0: [sda] Attached SCSI removable disk
</code>
确认无误后使用 [cci lang="bash"]cd /tmp/ && mkdir usb && mount /dev/sda1 usb[/cci] 尝试挂载ext4分区，一切正确的话没有任何输出，可以使用[cci lang="bash"]mount[/cci]来检查：
<code>
...
/dev/sda1 on /tmp/usb type ext4 (rw,relatime,user_xattr,barrier=1,data=ordered)
</code>
(注意: 如果是系统sysupgrade 上来的，此时有一个步骤必须做否则会遇到重启后无法正确挂载问题： [cci lang="bash"]rm /tmp/usb/etc/extroot.md5sum [/cci]，因为这个文件随着系统升级会发生变化，检查失败则OpenWRT会拒绝加载。

	<li>继续设置fstab来确保自动挂载喵~</li>
[cci]vi /etc/config/fstab[/cci]
<code lang="bash">
config global automount
        option from_fstab 1
        option anon_mount 1

config global autoswap
        option from_fstab 1
        option anon_swap 0

config mount
        option target   /overlay #注意这里
        option device   /dev/sda1
        option fstype   ext4
        option options  rw,sync
        option enabled  1 #这里
        option enabled_fsck 0

config swap
        option device   /dev/sda2
        option enabled  1 #如果有分配swap分区这里也打开
</code>
	<li>将 /overlay 下的所有文件复制到新挂载U盘中</li>
(注意: 如果之前有设置需要保留请注意操作，最好先将需要保留目录mv，拷贝overlay后再覆盖回去。
[cc_bash]cp -rf /overlay/* /tmp/usb/
reboot #确认好之后reboot察看自动加载分区，当然SSH需要重连
[/cc_bash]
正确加载的话，使用[cci_bash]df -h [/cci_bash]察看分区使用情况，注意/分区尺寸
<code>
Filesystem                Size      Used Available Use% Mounted on
rootfs                    6.5G    226.6M      6.0G   4% /
/dev/root                 1.5M      1.5M         0 100% /rom
tmpfs                    14.2M     88.0K     14.2M   1% /tmp
tmpfs                   512.0K         0    512.0K   0% /dev
/dev/sda1                 6.5G    226.6M      6.0G   4% /overlay
overlayfs:/overlay        6.5G    226.6M      6.0G   4% /
</code>
</ol>