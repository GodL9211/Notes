# Linux

：一个流行的操作系统。
- 发音为 `/ˈlɪnəks/` 。

## 历史

- 1991 年，芬兰人 Linus Torvalds 开发了 Linux 内核，将它按 GPL 协议授权为自由软件。
- Linux 内核与诸多 GNU 软件组合在一起，构成了一个操作系统，称为 GNU/Linux 。
  - Linux 在设计上借鉴了 Unix ，属于类 Unix 系统。但在社区的推动下，很快超越了 Unix 。
- 2007 年，开源发展实验室（Open Source Development Labs ，OSDL）与自由标准组织（Free Standards Group ，FSG）合并，成立 Linux 基金会，负责管理 Linux 社区。

## 发行版

- Linux 发行版（distribution）是将 Linux 内核和一些软件整合在一起的产品。
- 按照 GPL 协议，任何人都可以自定义 Linux 发行版，但是给别人使用时必须开源。不过有些 Linux 发行版中加入了一些收费软件，或者采用使用免费、服务收费的策略。

### Debian

- [官网](https://www.debian.org/)
- 发音为 `/dɛbiːjən/` 。
- 由 Debian 社区开发，于 1993 年开始发行。
- 采用 apt 作为软件包管理工具。

版本变更：
- Debian 8  ：2015 年发布，代号为 jessie  。
- Debian 9  ：2017 年发布，代号为 stretch 。
- Debian 10 ：2019 年发布，代号为 buster 。
- Debian 11 ：2021 年发布，代号为 bullseye 。

### Ubuntu

- [官网](https://ubuntu.com/)
- 中文名为 “乌班图” 。
- 基于 Debian 发行，于 2004 年开始发行。
- 由 Ubuntu 社区 开发，由 Canonical 公司赞助。
- 默认采用 GNOME 桌面系统，因为美观、易用而受桌面版用户欢迎。
- 版本号格式为 ` 年份尾号.月份 ` ，比如 2020 年 4 月发布了 20.04 LTS 版本。
  - 大概每隔半年发布一个版本。
  - 大概每隔 2 年发布一个 LTS（Long Term Support ，长期支持）版本。

版本变更：
- 12.04 LTS ：于 2012 年 4 月发布，代号为 precise 。
- 14.04 LTS ：于 2014 年 4 月发布，代号为 trusty 。
- 16.04 LTS ：于 2016 年 4 月发布，代号为 xenial 。
- 18.04 LTS ：于 2018 年 4 月发布，代号为 bionic 。
- 20.04 LTS ：于 2020 年 4 月发布，代号为 focal 。

### Mint

- [官网](https://www.linuxmint.com/)
- 基于 Ubuntu 发行，于 2006 年开始发行。
- 专为桌面用户设计，GUI 界面更人性化。

### Fedora

- [官网](https://getfedora.org/)
- 由 Fedora 社区开发，由红帽公司赞助。
  - Fedora 社区最初是为 Red Hat Linux 系统开发软件。2004 年该系统停止更新，Fedora 社区便在红帽公司的赞助下开始开发整个系统。
- 采用 yum 作为软件包管理工具，后来升级为 dnf 。
- 大概每隔半年发布一个版本。
  - 红帽公司会将一些新功能先添加到 Fedora 中，稳定之后再由 RHEL 继承。

### RHEL

：红帽企业版 Linux（Red Hat Enterprise Linux）
- [官网](https://access.redhat.com/products/red-hat-enterprise-linux/)
- 基于 Fedora 发行，于 2007 年开始发行。
- 由红帽公司开发。
- 可免费试用一段时间，付费之后才能正式使用，享受技术支持、版本升级。
- 大概每隔三年更新一个主版本，维护，比其它 Linux 发行版更加稳定、可靠，因此适用于服务器、工作站。

### CentOS

：社区企业操作系统（Community Enterprise Operating System）
- [官网](https://www.centos.org/)
- 基于 RHEL 发行。
  - 是对 RHEL 系统做出一些调整，比如去除商业软件，然后再发行。
- 由 CentOS 社区开发。
  - 2014 年，红帽公司雇佣了该社区的开发人员。
  - 2020 年底，宣布在 CentOS 8 之后停止发布新版本，转为开发滚动更新的 CentOS Stream ，作为 RHEL 的上游。

版本变更：
- CentOS 6
  - 于 2011 年发布。
  - 内核版本为 2.6.x 。
  - 集成了 Python 2.6 ，对应命令为 python 。
- CentOS 7
  - 于 2014 年发布，到 2024 年底停止维护。
  - 内核版本为 3.10.x 。
  - 用 systemd 进程代替 init 进程来初始化系统，用 systemctl 命令代替 service、chkconfig 来管理系统服务。
  - 默认的文件系统从 ext4 改为 xfs 。
  - 管理网络的工具从 ifconfig 改为 ip ，从 netstat 改为 ss 。
  - 管理防火墙的工具从 iptables 改为 firewall-cmd 。
  - 集成了 Python 2.7 ，对应命令为 python 。
- CentOS 8
  - 于 2019 年发布，到 2021 年底停止维护。
  - 内核版本为 4.18.x 。
  - 用 dnf 代替 yum 作为软件包管理工具，而 `/usr/bin/yum` 文件变成了指向 `/usr/bin/dnf` 的软连接。
  - 集成了 Python 3.6 ，对应命令为 python3 。

### openSUSE

- [官网](https://www.opensuse.org/)
- suse 的发音为 `/suːz/` 。
- 由 openSUSE 社区开发，由 SUSE 等公司赞助。
  - 2004 年，德国的 Novell 公司收购了 SUSE Linux 公司，将它改名为 openSUSE 并以开源形式发布。
  - 2010 年，Attachmate 集团收购了 Novell 公司，拆分成 Novell、SUSE 两个部门。
- 默认采用 KDE 桌面系统。
- 分为两种版本：
  - Leap ：常规版本
  - Tumbleweed ：滚动更新

### SLES

：SUSE 企业版（SUSE Linux Enterprise Server）
- [官网](https://www.suse.com/products/server/)
- 基于 openSUSE 发行。

### LFS

：Linux From Scratch ，一个自行构建 Linux 的项目。
- [官网](http://www.linuxfromscratch.org/)
- 提供了一些文档教程，讲解如何从网上下载 Linux 源代码，然后编译、安装。
- 常用于构建最简系统，可以只占几十 MB 磁盘。也有助于理解 Linux 的原理。

### Arch

：一个轻量级的 Linux 发行版。
- [官网](https://www.archlinux.org/)
- 由 Arch 社区开发，于 2002 年开始发行。
- 默认安装的是最简系统，只提供了命令行环境，需要用户自行添加软件、进行配置。因此使用门槛较高。
- 采用 pacman 作为软件包管理工具。
- 大部分软件采用滚动更新的方式，因此 Arch 没有划分版本号。
  - 更新时，先从旧版本更新到下一个版本。如果下一个版本兼容，则更新到再下一个版本。如果不兼容，则回滚到旧版本。
  - 几乎每周都有版本更新，能让用户体验到最新的版本，但也可能遇到最新的 bug 。

### Gentoo

- [官网](https://www.gentoo.org/)
- 由 Gentoo 社区开发，于 2002 年开始发行。
- 以 Portage 软件分发系统为核心，管理软件包。
  - 支持滚动更新。
- 支持高度的定制化。用户可以配置大部分软件，甚至可以自行从源代码编译软件。这方面与 LFS 类似。

### OpenWrt

：一个小型的 Linux 发行版，常用于嵌入式设备。
- [官网](https://openwrt.org/)
- 采用 opkg 作为包管理工具，其软件安装包的扩展名为 .ipk 。
