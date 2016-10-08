---
title: linux ssh 连接慢 超时中断 等问题解决
date: 2016-10-01 13:54:34
tags: linux ssh
---

### 解决超时问题

ssh 的配置文件在`/etc/ssh/sshd_config`下面，不知道不同的操作系统是不是回会有所不同。接下来研究下这个文件下又那些参数会导致ssh卡顿以及掉线的。

`ClientAliveInterval  60`这个个参数是指多少秒之后服务器向客户端发送一条消息默认是注释状态所以现在我们打开它。
`ClientAliveCountMax 3`示服务器发出请求后客户端没有响应的次数达到一定值就自动断开

上述两个修改后执行`service sshd reload`，新版本的linux貌似使用的systemd服务管理系统，那么命令就是`systemctl reload sshd`。其实都差不多

### 结局连接慢问题
1. DNS反向解析的问题
   `UseDNS no` 因为在ssh远程登录到服务器的时候，服务器会反向解析登   
   录的机器的IP的主机名引起的慢的问题
2. 关闭ssh的gssapi认证
   `GSSAPIAuthentication no`GSSAPI ( Generic Security Services 	
   Application Programming Interface) 是一套类似Kerberos 5 的通用网络安
   全系统接口。该接口是对各种不同的客户端服务器安全机制的封装，以消除
   安全接口的不同，降低编程难度。但该接口在目标机器无域名解析时会有问
   题



