title: Proxmox VE 单 公网IP 配置 NAT 和端口机端口映射
author: WCPE
tags: []
categories:
- Linux
- PVE
date: 2025-3-24 21:51:23
---

# 前言

有一台配置较高的服务器装上 PVE 系统放入机房托管，需要创建多个虚拟机使用

# 配置过程
开启 `IPV4`, `IPV6`转发

```
vim /etc/sysctl.conf
```

文件内写入以下配置

```bash
net.ipv4.ip_forward=1
net.ipv4.conf.all.rp_filter=1
net.ipv4.icmp_echo_ignore_broadcasts=1
net.ipv4.conf.default.forwarding=1
net.ipv4.conf.default.proxy_arp = 0
net.ipv4.conf.default.send_redirects = 1
net.ipv4.conf.all.send_redirects = 0
net.ipv6.conf.eno1.autoconf=0
net.ipv6.conf.eno1.accept_ra=2
net.ipv6.conf.default.forwarding=1
net.ipv6.conf.all.forwarding=1
net.ipv6.conf.default.proxy_ndp=1
net.ipv6.conf.all.proxy_ndp=1
```

接着继续配置 PVE 网卡文件信息

```
vim /etc/network/interfaces
```

```
auto lo
iface lo inet loopback

iface eno1np0 inet manual

auto vmbr0
iface vmbr0 inet static
	address 43.248.187.227/24
	gateway 43.248.187.1
	bridge-ports eno1np0
	bridge-stp off
	bridge-fd 0

# 物理网卡 vmbr0 配置一般不做改动
# 为虚拟机新建一个虚拟网桥 vmbr1
# 内网地址 192.168.100.1 虚拟机的网关
auto vmbr1         
iface vmbr1 inet static
	address  192.168.100.1
	netmask  255.255.255.0
	bridge-ports none
	bridge-stp off
	bridge-fd 0
	post-up echo 1 > /proc/sys/net/ipv4/ip_forward
	post-up echo 1 > /proc/sys/net/ipv4/conf/eno1/proxy_arp
	post-up   iptables -t nat -A POSTROUTING -s '192.168.100.0/24' -o vmbr0 -j MASQUERADE
	post-down iptables -t nat -D POSTROUTING -s '192.168.100.0/24' -o vmbr0 -j MASQUERADE

iface eno2np1 inet manual

iface eno3np2 inet manual

iface eno4np3 inet manual


source /etc/network/interfaces.d/*
```

重启网络服务

```
sudo service networking restart
```

安装 DHCP 服务(安装默认爆红无法启动，需要配置完成后才能启动)

```
apt install isc-dhcp-server
```

```
vim /etc/default/isc-dhcp-server
```

改成网卡

```
INTERFACESv4="vmbr1"
INTERFACESv6=""
```

接着修改配置

```
vim /etc/dhcp/dhcpd.conf
```

```
option domain-name "lan";
option domain-name-servers 223.5.5.5, 8.8.8.8, 1.1.1.1;

default-lease-time 600;
max-lease-time 7200;
subnet 192.168.100.0 netmask 255.255.255.0 {
    range 192.168.100.5 192.168.100.230;
    option subnet-mask 255.255.255.0;
    option domain-name-servers 223.5.5.5, 8.8.8.8, 1.1.1.1;
    option domain-name "lan";
    option routers 192.168.100.1;
    option netbios-name-servers 192.168.100.1;
    option netbios-node-type 8;
    get-lease-hostnames true;
    use-host-decl-names true;
    default-lease-time 600;
    max-lease-time 7200;
    interface vmbr1;
}
```

注意 `domain-name` 和 `domain-name-servers` 可以和母机一直以防止 DNS 无法链接

通过 `cat /etc/resolv.conf` 在母机上查询 DNS 配置

通常返回以下结果
```
root@pve:~# cat /etc/resolv.conf
search lan
nameserver 223.5.5.5
nameserver 8.8.8.8
nameserver 1.1.1.1
```

重启 DHCP 服务
```
systemctl restart isc-dhcp-server
```

查询 DHCP 服务状态
```
systemctl status isc-dhcp-server
```

安装 rinetd

```
apt install rinetd
```

使用 iptables 配置端口映射

```
iptables -t nat -A PREROUTING -p tcp -m tcp --dport 30022 -j DNAT --to-destination 192.168.100.5:22
```

删除只需要把新增映射的 -A 改成 -D 即可

```
iptables -t nat -D PREROUTING -p tcp -m tcp --dport 30022 -j DNAT --to-destination 192.168.100.5:22
```

查看NAT规则，并显示行号
```
iptables -t nat --list --line-number
```

至此已经可以正常使用了