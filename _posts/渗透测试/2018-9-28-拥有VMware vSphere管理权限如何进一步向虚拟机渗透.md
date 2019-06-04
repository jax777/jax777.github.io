---
layout: post
title: 拥有VMware vSphere管理权限如何进一步向虚拟机渗透
categories: 渗透测试
tag: vSphere
---

实际测试中遇到VMware vSphere构建的云环境，同时在已获取域管理员账号的情况下，很大可能可以直接用域管理员登录vsphere（多数vSphere会配置域管理员用户登录），这时就需要尝试对vSphere管理的所有虚拟机进行进一步渗透。

### VMware vSphere 简介

[VMware vSphere 官网链接](https://www.vmware.com/products/vsphere.html)

vSphere: The Efficient and Secure Platform for Your Hybrid Cloud

![](/styles/images/2018/vSphere.png)

VMware vSphere  在个人理解中就是管理一群exsi主机和一群运行在主机之上vm的一个产品 。
通过vSphere可以获得开关虚拟机、复制虚拟机的常规权限。

vSphere client 界面 类似常用的vmware workstation 的功能 （其实vmware workstation 也可以做vSphere客户端用，远程连接vsphere即可）

![](/styles/images/2018/vSphere1.png)

![](/styles/images/2018/vSphere2.png)

### 向虚拟机内部渗透

当前所有权限如下

- 利用客户端开启console （如果有密码 可以尝试登陆了）
- 开关机、调整虚拟机配置
- 克隆虚拟机
- 复制移动exsi主机上的文件，例如虚拟机的 vmdk 虚拟磁盘文件，还有一些存放于主机上的iso 光驱镜像文件

*在目标机器没有加入域且不知道账户的情况下，只能尝试通过文件系统进入目标了，通过挂载目标vm的vmdk来获取账户或者直接提取敏感文件。*

当前正在运行的vm的虚拟磁盘文件是不可以再次挂载的，可以通过**热克隆**目标vm，再对克隆出的vmdk进行挂载，达到对文件系统的控制

**读取vmdk的几种方法**

- 可以尝试通过一个已控的vm将目标vmdk挂载，进入当前已控vm读取目标文件信息即可。（一般的linux的vmdk 只能找个linux 的vm 进行挂载）
- 找不到可控vm时可以直接拿目标平台搭建一个，系统iso文件一般查看目标vm的光驱挂载即可找到，常人装完系统都不会把光驱挂载给改了吧，毕竟装完都用不上了。
- 理论上也通过vmware 提供的[vvdk (Virtual Disk Development Kit)](https://code.vmware.com/web/sdk/6.7/vddk) 远程挂载 ，vvdk 5.1 版本提供了 vmware-mount 文件可用来直接远程挂载，vvdk 5.5 版本以后不再提供vmware-mount 但是可以直接编译提供的vixdisklibsample(远程挂载最终在实验环境没有尝试成功。。哪位师傅成功了教教我)


最后给个**胆子大、不要命的方法**

- 直接重置被管理的exsi密码，在exsi上做中间人，其实也是可以获取虚拟机流量的，运气好就渗透进去了。

### 最后

[《云安全原理与实践》](https://yq.aliyun.com/articles/212736)