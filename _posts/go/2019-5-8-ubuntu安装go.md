---
layout: post
title: ubuntu 下go 安装环境配置
categories: go
tag: 环境安装
---

# ubuntu 系统 go语言安装、配置

## 下载

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

## 设置代理下载

> https://github.com/goproxy/goproxy.cn/blob/master/README.zh-CN.md
