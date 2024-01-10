---
title: "编译一个支持Infiniband的OpenWRT固件"
date: 2024-01-10T20:42:11+09:00
draft: false
tags: []
categories: ["教程"]
isCJKLanguage: true
toc:
  enable: true
  auto: true
---

# 前言

我有一台服务器，我有一堆 `Mellonx ConnectX 2/3` 的 IB 卡，我还有一台 IB Switch，以及 100Gbps 的光模块若干喵……（以上全部是捡垃圾来的喵）

于是，我现在需要一个 OpenWRT 软路由，同时支持高速 IB 网络喵！

## 准备工作

咱的编译环境是 `Arch Linux`，但是理论上任何 Linux （包括 OpenWRT 系统自身）都可以编译喵~唯一需要注意的是各种依赖库和软件的编译（特别是 Host 端）

好在 OpenWRT 的 Wiki 很详细，找到自己系统对应的指令照着复制粘贴上去就好了喵：

https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem#archmanjaro

Arch Linux 的指令：

```bash
# Essential prerequisites
pacman -S --needed base-devel autoconf automake bash binutils bison \
 bzip2 fakeroot file findutils flex gawk gcc gettext git grep groff \
 gzip libelf libtool libxslt m4 make ncurses openssl patch pkgconf \
 python rsync sed texinfo time unzip util-linux wget which zlib
 
# Optional prerequisites, depend on the package selection
pacman -S --needed asciidoc help2man intltool perl-extutils-makemaker swig
```

注意点：

1. `Python2` 包不是必须的（除非你想要编译一个非常古老的 OpenWRT 版本）
2. 编译过程中，可能会突然出现 `XXX command/library not found` 之类的提示，然后整个 Make Error，这个时候找到对应包装上继续编译步骤就好喵~

## 编译过程

0. 确认准备工作中的所有包已经正常安装并可以正确执行；准备一个起码有 40G 空余空间的卷；
1. 克隆 OpenWRT 的源代码并准备编译，之后指令如无特别说明均基于该 `openwrt` 目录执行：
   
```bash
git clone https://git.openwrt.org/openwrt/openwrt.git
cd openwrt
git checkout master
git pull
./scripts/feeds update -a
./scripts/feeds install -a
```

2. 找到目标系统的默认配置文件并替换 `.config` 文件。 OpenWRT 的所有编译配置均可在其下载目录中直接找到，这里因为是给 x86_64 虚拟机使用的，直接用 x86_64 配置喵：

```bash
wget https://downloads.openwrt.org/snapshots/targets/x86/64/config.buildinfo -O .config
make defconfig
```

3. 配置 OpenWRT  `make menuconfig`

    这里可以选择各种打包参数（默认会全部镜像都打包，但是其实不需要，选择一个适合的就好），建议把 `luci` 相关的包配置进镜像，这样可以启动了就有 Web 配置页面用（不配置启动后直接SSH进去用 opkg 装也OK喵）

4. 编译 `llvm-bpf`（如果在配置阶段选择自行编译的话，如果你的系统里就有可以考虑在配置里选择使用 Host 的）
   
```bash
# llvm-bpf 补丁
mkdir tools/llvm-bpf/patches
wget https://va1der.ca/~public/openwrt/patches/llvm-bpf/901-fix-fuzzer-linking.patch -O tools/llvm-bpf/patches/901-fix-fuzzer-linking.patch
wget https://va1der.ca/~public/openwrt/patches/llvm-bpf/902-use-libatomic-as-required.patch -O tools/llvm-bpf/patches/902-use-libatomic-as-required.patch

# 编译 llvm-bpf
make -j $(nproc) V=sc tools/llvm-bpf/compile
```

注意：根据 Master 和各个开源社区的PR合并，布丁可能过期，如果编译的时候被提示补丁无法应用，删除对应文件重试 `make` 即可喵。这个步骤会消耗比较多的CPU和内存。

5. 编译其它工具，同样需要注意 Error 输出喵~

```bash
make -j $(nproc) V=sc toolchain/install
```

6. 下载源代码，如果网络不错的话几分钟就好喵~

```bash
make download
```

7. 给 perl 打布丁

```bash
( cd feeds/packages/lang/perl && wget https://va1der.ca/~public/openwrt/patches/perl/perl_fix_memmem_and_segfault.patch -O - 2> /dev/null | patch -p 1 )
```

8. 配置 Kernel ：`make -j$(nproc) kernel_menuconfig`，注意需要在 `Drivers` 里选择 `Infiniband` 支持模块，同时要在 `Ethernet` 里选择 `mlx4/mlx5` ，mlx4 支持 ConnectX 3 / Pro 及之前更古老的网卡，mlx5 则支持 ConnectX 4 及更新的网卡；

9.  最终编译 ~~蓝蓝路~~ `make -j 1 V=sc world` 注意这里因为有工具链会使用 `ninja` 等构建工具，不建议并发开太多喵~

## 镜像应用

1. 编译好的镜像和包都在 `bin/target/x86/64/` 目录下，直接按照官方镜像运用方式启动即可
2. 可以使用 nginx/darkhttp 等启动一个临时 http 服务，然后将自己编译的 OpenWRT 目录挂上去，变成自定义镜像站，这样可以 opkg install 到当前编译 Kernel 支持的 mods ; 如果 opkg update 提示签名错误，则将编译目录下生成的签名.pub添加到 `/etc/opkg/keys/` 里即可
3. ib 相关支持的 kernel mod 目前 似乎不会自动集成到 opkg 包里变成 kmod-xxx 包，解决办法是直接在 build 目录里搜索 `ib_core.ko` `ib_cm.ko` `ib_ipoib.ko` `ib_umad.ko` `ib_uverbs.ko` `mlx4_core.ko` `mlx4_en.ko` `mlx4_ib.ko` 等文件拷贝到 `/lib/modules/<kernel version>/` 目录中
4. 编辑 `/etc/modules.d/30-hw-network` 文件，加入以下内容并链接到 `/etc/modules-boot.d` 以自动加载 ib 相关模块：

```
ib_core
ib_ipoib
mlx4_core
mlx4_en
mlx4_ib
```
5. 建议安装 `rdma` 包用来控制 ib 相关 rdma 配置。
6. 如果模块加载和配置均成功， ipoib 的网卡会自动出现在 Device 里，按照普通网卡配置 interface 即可，注意不可以将其加入到桥接，因为它是个 L3 的协议喵

# 后记

全文仅凭记忆杂凑而成，如有任何疏漏或者错误记录，欢迎留言告知喵~ 

以下项目是咱参考过的弯路，仅供参考喵：

1. https://github.com/allegro0132/Openwrt-mlnx-ofed/ 该项目使用的代码过于老旧，尤其是 Kernel 5.x 之后大量API变更导致编译不可能喵。
2. https://network.nvidia.com/products/infiniband-drivers/linux/mlnx_ofed/ //因为内核里已经有驱动了，所以也不需要手动去下载这个驱动加入编译喵。