---
title: "Debian 10 iptables防火墙常用命令"
date: 2023-04-07T10:09:29+08:00
draft: false
categories: ["其他"]
---

# 查看某条规则下的防火墙开放端口

```shell
#查看全部规则
iptables -L --line-numbers
#查看ufw-user-input 规则
iptables -L ufw-user-input --line-numbers 
```

# 授权开放防火墙端口

```shell
iptables -I INPUT -p tcp --dport 795 -j ACCEPT
```
# 授权关闭防火墙端口

```shell
iptables -D INPUT -p tcp --dport 8888 -j ACCEPT
```
# 删除某条规则

```shell
iptables -D ufw-user-input 6
```
# 查看正常服务的对外端口

```shell
netstat -tlpn
```

# 保存防火墙规则

如果没有安装iptables-persistent则先安装iptables-persistent

```shell
sudo apt-get install iptables-persistent
```
输入以下命令保存规则持续生效

```shell
netfilter-persistent save
netfilter-persistent reload
```
