---
title:   关于wordpress更换端口之后的问题以及mysql配置和python串口通讯
date: 2016-10-03 20:23:26
tags: mysql nginx 工作日志 python wordpress
---

# 2016年10月12日16:03:57
<!-- vscode-markdown-toc -->
* 1. [第一个问题](#-0)
* 2. [可能会出现的问题](#-1)
* 3. [Mysql常用命令](#Mysql-2)
* 4. [wordpress常识](#wordpress-3)

<!-- /vscode-markdown-toc -->
```
python -m serial.tools.list_ports #查看本机串口命令

mysql -uroot -p1qazxsw2#本机mysql目录

/usr/local/nginx/conf/vhost #nginx目录

/etc/my.cnf#mysql配置文件

flush privileges; #刷新数据库设置

GRANT ALL PRIVILEGES ON shandong.* TO 'demo'@'%'WITH GRANT OPTION 

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '1qazxsw2' WITH GRANT OPTION

$P$Bs1WpqdRPP8CeLYkA3QwcC5ANOSzN3.
```

<b> URL重写  rewrite<b>

##  1. <a name='-0'></a>第一个问题
>修改nginx配置文件 修改域名检查配置文件是否正确 nginx testconfig(待定)检测配置文件是否正确
>查看wordpress配置文件`wp-config.php`中数据库连接名和账号密码
>连接数据库 修改表wp-option中site-url 和 home字段与nginx中设置的域名一至


##  2. <a name='-1'></a>可能会出现的问题
>数据库没有给外部网络的访问权限
>解决方法:`GRANT ALL PRIVILEGES ON *.* TO '<你的用户名>'@'%' IDENTIFIED BY '<你的密码>' WITH GRANT OPTION`
>看不懂查 SQL语句 GRANT
>去看防火墙3306端口是否开放。简单粗暴的方法就是关闭防火墙
>一般的linux系统使用的是iptables防火墙，Centos 7以上使用的是firewall 其命令是firewall-cmd --add-port=3306/udp
>实在不会使用ssh登陆远程主机 输入命令 mysql -u用户名 -p密码 注意不要有空格

##  3. <a name='Mysql-2'></a>Mysql常用命令
>show databases;#查看当前mysql中所有的数据库
>use 数据库名;#进入数据库
>show tables;查看当前数据库中所有表
>select * from 表;#查看表中所有的数据
>update set 字段 = 值 where 条件;

##  4. <a name='wordpress-3'></a>wordpress常识
>主题目录`wp-content\themes`
>插件目录`wp-content\wp-plugins`
>配置文件`wp-config.php`