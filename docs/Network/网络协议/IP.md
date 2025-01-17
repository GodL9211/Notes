# IP

TCP/IP 协议簇包含 IP、TCP、UDP 等多种协议，是因特网的核心通信协议。

## IP 协议

：因特网协议（Internet Protocol）
- 属于网络层协议，规定了网络上的主机用 IP 地址确定自己的身份。
- 面向无连接，只提供尽力而为的数据传输服务 —— 只管把数据发送出去，不进行差错控制。
  - 是不可靠的通信协议。
- 通过 IP 协议，主机可以加入一个私有网络或因特网。
  - 私有 IP ：接入私有局域网的每台主机都必须被分配一个该局域网的 IP 地址。
    - 一个主机可以拥有同一个网络的多个 IP 地址，也可以拥有不同网络的多个 IP 地址。
  - 公有 IP ：接入因特网的每台主机都必须被分配一个公有 IP 地址。
- IP 协议主要有 IPv4、IPv6 两个版本，后者尚未普及。

## 相关概念

- IP 数据包中会记录源 IP 地址、目标 IP 地址。封装成数据帧之后，记录的就是源 Mac 地址、目标 Mac 地址。
  - 数据链路层的通信结点收到一个数据帧之后，会先检查是否有错误，然后查看数据帧的目标地址，如果目标地址是本地地址或广播地址就接收该帧，否则转发。
- 目标 IP 地址的分类：
  - 单播地址：发往某个指定的 IP 地址。
  - 多播地址：发往一组 IP 地址。
  - 广播地址：地址全为 1 ，可以被所有结点接收。
- 最大传输单元：Maximum Transmission Unit（MTU），允许传输的 IP 数据包的最大大小，单位 Bytes 。
  - 这是数据链路层的概念，主要由网卡决定。
  - 以太网中网卡的 MTU 通常设置为 1500 ，因此经常会将一个 TCP 数据段拆分成多个 IP 数据包再传输。
  - 如果某台主机的 MTU 比路由器的 MTU 大，过大的数据包就会被路由器拆分传输，导致数据包数量变多。

## IPv4

IPv4 地址的长度为 32 位，分为 4 个字段，每个字段有 8 位，用点分十进制表示就是每个字段有一个 0~255 的十进制数。
- 采用“网络号+主机号”的结构，每个网络号表示一个逻辑网络。

根据开头几位的不同，可将 IPv4 地址分为 5 类：
- A 类地址
  - 以一个 0 开头，网络号占 8 位，主机号占 24 位，取值范围为 `0.0.0.0~127.255.255.255` 。
  - 第一个字段有 2^7=128 个可能值，取值范围为 0~127 。
  - 排除网络号全为 0 的 IP 地址、私有地址 `10.*.*.*`、环回地址 `127.*.*.*` 之后，可分配的网络号有 125 个。
    - 每个网络排除全为 0、全为 1 的主机号之后，可分配的主机号有 224-2 个。
    - 因此，可分配的 A 类地址总共有 231 个左右，占全部 IP 地址的一半。
  - A 类的私有地址：`10.*.*.*`
- B 类地址
  - 以 10 开头，网络号占 16 位，主机号占 16 位，取值范围为 `128.0.0.0~191.255.255.255` 。
  - 第一个字段有 2^6=64 个可能值，取值范围为 128~191 。
  - 可分配的网络号有 214 个，每个网络可分配的主机号有 216-2 个。
  - B 类的私有地址：`172.16.*.*~172.31.*.*` ，共 16 个网络号。
- C 类地址
  - 以 110 开头，网络号占 24 位，主机号占 8 位，取值范围为 `192.0.0.0~223.255.255.255` 。
  - 第一个字段有 2^5=32 个可能值，取值范围为 192~223 。
  - 可分配的网络号有 221 个，每个网络可分配的主机号只有 28-2=254 个。
  - C 类的私有地址：`192.168.*.*` ，共 256 个网络号。
- D 类地址
  - 以 1110 开头，用于多播地址，取值范围为 `224.0.0.0~239.255.255.255` 。
- E 类地址
  - 以 11110 ，保留给实验或未来使用，取值范围为 `240.0.0.0~247.255.255.255` 。

以下 IP 地址不能用于因特网：
- 私有 IP 地址  ：保留给私有网络使用。这样当主机接入因特网时，路由器能识别出该 IP 是私有 IP 从而忽视它。
- `127.*.*.*` ：环回地址（lookback、localhost）。
  - 发向该 IP 地址的数据包会直接回送到本机，不会被网卡发出去。即只在物理层传输，不会到数据链路层。
  - 127.0.0.0 是网络号，一般使用 127.0.0.1 。
  - 环回地址对应的子网掩码为 255.255.255.255 。
- `0.0.0.0` ：在路由表中用作默认网关，没有找到路由的数据包都会转发给默认网关。
  - 如果服务器监听 0.0.0.0 ，则会接受来自任何 IP 地址的访问。

每个网络的以下地址有特殊用途：
- 主机号全为 0  ：表示该网络本身的网络号。
- 网络号全为 0  ：表示只发送给该网络的主机。
  - 数据包会被路由器限制在该网络内，发送给主机号对应的主机。
- 主机号全为 1  ：直接广播地址，数据包会被广播给目标网络内的所有主机。
- IP 地址全为 1 ：受限广播地址，数据包会广播给当前网络内的所有主机。
  - 对应的 MAC 地址为 ffff.ffff.ffff 。
- 一般将每个网络的第一个主机号留给网关使用，比如 10.0.0.1 。

### IPv4 短缺问题

IPv4 地址短缺的问题越来越严重，原因包括：
- B 类地址的一个网络中可分配的主机号太多，实际接入的主机少，即使接入了这么多主机也会因为路由表太大增加路由器的负担，导致网络服务质量下降。
- C 类地址可分配的网络号很多，但每个网络中可分配的主机号较少，只适用于小型局域网。

为了解决 IPv4 地址短缺的问题，人们采用了以下对策：
- 划分子网（subnet）
  - ：将一个容量大的网络分成几个子网络，将 IP 地址改成 ` 网络号+子网号+主机号 ` 的三级结构（只适用于 A、B、C 类地址），然后用 `IP 地址 子网掩码 ` 或 ` IP 地址/子网掩码长度 ` 的形式表示该子网。
    - 网络号和子网号对应位的子网掩码（subnet mask）全为 1 ，主机号对应位的子网掩码全为 0 。
    - 一个子网 IP 地址与其子网掩码按位与的结果，就是其子网号。
  - 例如：对于 192.168.0.0 ，该网络可容纳 2^8-2 个主机，
    - 从 8 位主机号中取出前一位，便可划分 192.168.0.0/25 和 192.168.0.128/25 两个子网，每个子网可容纳 2^7-2 个主机。
    - 再将 192.168.0.128/25 的子网掩码延长一位，又可划分两个子网，每个子网可容纳 2^6-2 个主机。
    - 每个网络中全为 0 和全为 1 的主机号不可用，因此在 192.168.0.128/25 这个子网中，第一个可用的 IP 地址是 192.168.0.129/25 ，最后一个可用的 IP 地址是 192.168.0.254/25 。
  - 不划分子网时，A 类地址的默认子网掩码是 255.0.0.0 ，B 类地址的默认子网掩码是 255.255.0.0 ，C 类地址的默认子网掩码是 255.255.255.0 。
  - 子网掩码为 255.255.255.255 表示该子网只有一个主机，通常用于环回网口。
- 构成超网（supernet）的无类别域间路由（CIDR）技术
  - ：将网络号前 n 位相同的网络看作同类并合并为一个网络，又称为地址聚合、路由聚合。
  - 例如：192.168.0.129 与 192.168.0.130 的前 30 位相同，聚合后的网络为 192.168.0.128/30 。
  - 例如：汇聚 .01010000 与 .01010001 、 .01010010 时，三个 IP 地址的前 30 位相同，但是只有两位主机号，去掉全为 0 和全为 1 的主机号后只能分配两个主机号，所以聚合后的网络应该为 .01010000/29 。又因为 .01010000 这个地址已经被占用，为了避免发生冲突，要一直退位，所以聚合结果为 .01000000/27 。
- 网络地址转换（Network Address Translation ，NAT）
  - ：平时给主机分配私有 IP ，当主机联网时才换成公有 IP 地址。
  - 该技术多用于拨号上网：拨号时主机被 ISP 分配一个公有 IP 地址，联网结束时被收回。
- 使用 IPv6 地址
  - 能从根本上解决 IPv4 地址短缺的问题。
  - 不过全面更换 IP 地址需要的时间久，导致 IPv6 地址还不能通用。

## IPv6

- IPv6 地址的长度为 128 位，分为 8 个字段，每个字段有 16 位，用冒号十六进制表示就是每个字段有 4 个 0~F 的十六进制数。
- 每个字段前面的 0 可以省略，不过每个位段至少要有一个数字，或者连续几个位段全为 0 时可以省略它们而用双冒号表示。
  - 双冒号不可以出现两次，否则无法判断省略的位数。
  - 例如：0000:210A:0000:0000:02AA:000F:0000:0000 可简写成 0:210A:0:0:2AA:F:0:0 ，再进一步可简写成 0:210A::2AA:F:0:0 或 0:210A:0:0:2AA:F::。
- IPv6 协议具有巨大的地址空间、新的协议格式、有效的分级结构等优点。
