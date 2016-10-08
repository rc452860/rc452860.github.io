---
title: 关于VPN搭建的一些问题
date: 2016-09-26 22:29:19
tags: VPN PPTP
---
在vultr上购买了一个日本节点的vps搭建vpn服务器。之前也有用其他服务器搭建过vpn。但是今天一直没弄成功。
### 虚拟化技术

搭建pptp服务器其实很简单。首先确定你购买的服务器是属于那种虚拟技术的

| 虚拟化类型  | 特性                           |
| ------ | ---------------------------- |
| Xen    | 半虚拟化技术 可以加载内核模块 内存独占 硬盘小 带宽小 |
| OpenVz | 类似于Docker容器 无法升级内核           |
| Kvm    | 完全虚拟化可以运行各种类型的操作系统包括windows  |


具体的大家可以去看 [VPS虚拟化产品 - Xen、OpenVZ、KVM、Hyper-V、VMWare虚拟化技术介绍](http://www.xshell.net/linux/1491.html)

这三种虚拟类型都是可以安装pptp的但是有些厂家把Tun/Tap给禁用了。自己可以去看看有的面板是可以打开的

```shell
cat /dev/ppp
cat: /dev/ppp: No such device or address    #返回这个说明PPP已经开启。
cat /dev/net/tun
cat: /dev/net/tun: File descriptor in bad state    #返回这个说明Tun/Tap已经开启。
```

然后确定自己vps是符合安装pptp要求的话那么就可以通过系统自带的包管理工具下载PPTP了

### 下载安装pptp
比如：ubuntu使用`apt-get install -y pptp` centos使用`yum install -y pptp`

一般的包管理工具会自动去解析依赖项把ppp给装上

### 关于pptp协议
pptp是基于ppp上面第一个协议。具体的去看[rfc 2637](http://www.ietf.org/rfc/rfc2637.txt)

### 配置文件
完了就要开始配置配置文件了，一般会有三个文件给你配置`/etc/pptpd.conf` `/etc/ppp/options.pptpd` `/etc/ppp/chap-secrets`

配置`/etc/pptpd.conf`守护进程设置，一般配置本地IP和用户IP段就好了。具体的可以看配置文件中的注释

```
option /etc/ppp/options.pptpd
logwtmp
localip 192.168.80.1
remoteip 192.168.80.101-200
```

配置PPTP设置 `/etc/ppp/options.pptpd` 配置名字 使用的加密方法 以及dns还有一些其他选项。具体还是去看配置

```
name pptpd
refuse-pap
refuse-chap
refuse-mschap
require-mschap-v2
require-mppe-128
proxyarp
lock
nobsdcomp 
novj
novjccomp
nologfd
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

最后一个文件是设置密码的`/etc/ppp/chap-secrets`
```
#格式： 用户名 pptpd 密码 ip地址（*代表随机）
username  pptpd   passwd  *
```

剩下的是防火墙问题。最简单是关闭防火墙。`service iptables stop`，或者添加防火墙端口开放。
PPTP主要开放如下几个端口
1723 TCP
43   TCP/UDP
53   UDP(这个是DNS端口吧。貌似不开连上去上不了网)

### iptables 防火墙配置
配置 iptables 设置，允许 PPTP 客户端访问：
```
iptables -A INPUT -i ppp+ -j ACCEPT
iptables -A OUTPUT -o ppp+ -j ACCEPT

iptables -A INPUT -p tcp --dport 1723 -j ACCEPT
iptables -A INPUT -p 47 -j ACCEPT
iptables -A OUTPUT -p 47 -j ACCEPT
iptables -A INPUT -p 54 -j ACCEPT
iptables -A OUTPUT -p 54 -j ACCEPT

iptables -F FORWARD
iptables -A FORWARD -j ACCEPT

iptables -A POSTROUTING -t nat -o eth0 -j MASQUERADE
iptables -A POSTROUTING -t nat -o ppp+ -j MASQUERADE
```
使用下面命令保存新的 iptables 规则：
``` rc.d save iptables```
systemd 用户需要使用：
```iptables-save > /etc/iptables/iptables.rules```


### ufw 防火墙配置
在 /etc/default/ufw中，修改默认的转发规则
```
/etc/default/ufw
DEFAULT_FORWARD_POLICY=”ACCEPT”
修改/etc/ufw/before.rules，添加如下配置到 *filter 之前
/etc/ufw/before.rules
# nat Table rules
*nat
:POSTROUTING ACCEPT [0:0]

# Allow traffic from clients to eth0
-A POSTROUTING -s 172.16.36.0/24 -o eth0 -j MASQUERADE

# don.t delete the .COMMIT. line or these nat table rules won.t be processed
COMMIT
```


代开端口 1723：
```ufw allow 1723```
重启ufw
```
ufw disable
ufw enable
```
还有Centos7 使用的是自己的firewall，firewall的使用`firewall-cmd`来控制防火墙。如果使用centos7的话可以自行百度。也可以换成iptables

哦对了还要开启IP转发
### IP转发
在`/etc/sysctl.conf`中修改或添加下面这段
`net.ipv4.ip_forward=1`

### 服务启动
关于服务的启动方式分两种个暂时我就只试过两种
原先的service启动方式 
`service pptpd start`
后来linux改成systemd的服务管理器。相关资料去看[systemd](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)
`systemctl start pptpd.service`
[20$优惠连接](http://www.vultr.com/?ref=6993197-3B)
[10$连接](http://www.vultr.com/?ref=6992791)