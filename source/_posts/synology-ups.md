---
title: 群晖接入 UPS 实现断电自动关机
author: WCPE
date: 2024-07-04 23:04:00
categories:
- 记录
tags:
- '群晖'
- 'Synology'
---

# 前言

最近乘着回家赶着把之前物理机装的黑群晖倒腾一下升级一下硬件，原先是物理机直接安装 `DSM6` 引导也是旧的升级很不方便。

然后我就给物理机装了个 `Proxmox Virtual Environment` 并将黑群晖虚拟化，之前物理机黑群晖并没有 UPS 现在想给它配个 UPS于是我就在京东购买了一款 APC 的 UPS 就是这个 `APC BK650M2_CH` 后备式 UPS 功率输出满足我的机器。

但是它只有一个通信口，我还有另外一台服务器需要接入 UPS 并不能直接将 ~~USB 通信口直通到黑群晖~~了(好像 PVE 也得关机)，只能用笨一点的办法，轮询 Ping 网关了并且将软路由不接入 UPS 直接使用市电来达到一个断电断网的效果，然后在接入 UPS 的服务器上配置定时检查网关存活的脚本定时执行

# 过程

我是秉着一个能不自己动手就不自己动手的原则，先上搜索引擎搜索，是很多大神都有现成的代码，我先是搜索到了这篇帖子 [群晖设置Ping不通路由器自动关机 实现断电自动关机](https://i4t.com/16275.html) 

从中得到了配置的脚本

```bash
#!/bin/sh
MonitorIP=192.168.31.1
DelayTime=180s
if ping $MonitorIP -W 2 -w 2 -c 2 | grep '^[0-9].*ms$' > /dev/null
then
echo "Power on."
else
synologset1 sys warn 0x11600036
sleep $DelayTime
if ping $MonitorIP -W 2 -w 2 -c 2 | grep '[0-9].*ms$' > /dev/null
then
synologset1 sys warn 0x11600035
else
synologset1 sys warn 0x11600037
poweroff
fi
fi
exit 0
```
我直接丢入系统执行，发现无论 IP 是否存活都会进到 `Power on.` 也就是说正则匹配不上，没办法了，还是得自己动手，丰衣足食

然后我尝试 debug 输出这个 ping 匹配的值
```bash
echo ping $MonitorIP -W 2 -w 2 -c 2 | grep '^[0-9].*ms$' > /dev/null
```
发现返回值一直为空 我手动执行了 ping 指令

---

- 能 Ping 通
```
root@MyNas:~# ping 192.168.1.1 -W 2 -w 2 -c 2
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=64 time=0.344 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=64 time=0.267 ms

--- 192.168.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.267/0.305/0.344/0.042 ms
```

- 不能 Ping 通
```
root@MyNas:~# ping 192.168.1.123 -W 2 -w 2 -c 2
PING 192.168.1.123 (192.168.1.123) 56(84) bytes of data.

--- 192.168.1.123 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 999ms

```
---

发现使用正则 `^[0-9].*ms$` 根本无法匹配上 于是我就稍微改动了一下脚本

# 结果
```bash
#!/bin/sh
MonitorIP=192.168.1.1
DelayTime=60s
# 发送ping命令并检查其返回值
if ping -c 2 -W 2 $MonitorIP > /dev/null; then
    echo "网关存活 停止关机操作."
else
    synologset1 sys warn 0x11600036
    echo "网关不通 等待 $DelayTime 再次确认关机."
    sleep $DelayTime
    if ping -c 2 -W 2 $MonitorIP | grep -q "100% packet loss"; then
        synologset1 sys warn 0x11600037
        echo "网关不通 开始关机!"
        poweroff
    else
        synologset1 sys warn 0x11600035
        echo "网关存活 停止关机操作."
    fi
fi
exit 0
```

目前放系统上每分钟一次执行正常运行中