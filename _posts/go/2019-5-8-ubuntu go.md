---
layout: post
title: ubuntu 下go 环境配置 + vscode 远程调试
categories: go
tag: 环境安装
---

## ubuntu 系统 go语言安装、配置

### 下载

- https://golang.org/dl/ 查找linux 压缩包

    ` curl -O https://dl.google.com/go/go1.12.5.linux-amd64.tar.gz`
    
    解压
    `tar  -C /usr/local zxvf go1.12.5.linux-amd64.tar.gz `

- 配置路径

    `vim /etc/profile`

    加入

    `export PATH=$PATH:/usr/local/go/bin`

- 设置GOPATH
    工作目录，存放第三方包以及自己的代码

    `vim /etc/profile`

    加入

    `export GOPATH=/home/gopath`

### 设置代理下载

> https://github.com/goproxy/goproxy.cn/blob/master/README.zh-CN.md

### go mod 包管理

```bash
go mod init packagename 

go.mod 提供了module, require、replace和exclude 四个命令

    module 语句指定包的名字（路径）
    require 语句指定的依赖项模块
    replace 语句可以替换依赖项模块
    exclude 语句可以忽略依赖项模块

```
目录下会生成 go.mod  go.sum

## vscode

### 安装

- 安装扩展 Remote - SSH
- 配置ssh 文件
  ![](/styles/images/2019-8/vscodessh.png)
config 文件
```
# Read more about SSH config files: https://linux.die.net/man/5/ssh_config
Host 49.231.1.1.
    HostName 49.231.1.1.
    User ubuntu
```
- id_rsa 私钥置于该目录下

### 使用
![](/styles/images/2019-8/vscoderemote.png)

- go 远程调试