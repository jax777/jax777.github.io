---
layout: post
title: git 设置socks代理(众所周知的原因github clone 十分缓慢)
categories: git
tag: git
---

# 说明

众所周知的原因，从github  上 clone 项目十分缓慢

## git 使用 socks 代理

```bash
# 之前
$ git clone https://github.com/jax777/nmap.git
Cloning into 'nmap'...
remote: Enumerating objects: 116, done.
remote: Counting objects: 100% (116/116), done.
remote: Compressing objects: 100% (86/86), done.
Receiving objects:  17% (13164/73323), 8.05 MiB | 11.00 KiB/s

# 设置
$ git config --global http.proxy 'socks5://127.0.0.1:1080'

$ git config --global https.proxy 'socks5://127.0.0.1:1080'

# 之后


$ git clone https://github.com/jax777/nmap.git
Cloning into 'nmap'...
remote: Enumerating objects: 116, done.
remote: Counting objects: 100% (116/116), done.
remote: Compressing objects: 100% (86/86), done.
remote: Total 73323 (delta 45), reused 55 (delta 30), pack-reused 73207
Receiving objects: 100% (73323/73323), 86.38 MiB | 888.00 KiB/s, done.
Resolving deltas: 100% (56099/56099), done.
Checking out files: 100% (2449/2449), done.

# 关闭代理

off)
git config --global --unset http.proxy
git config --global --unset https.proxy

# 查看代理状态
status)
git config --get http.proxy
git config --get https.proxy
```
