---
layout: post
title: wsl win10 linux 子系统
categories: wsl
tag: tips
---

## wsl linux文件 在win10中打开

- wsl 中 /mnt 下会挂载 win10中各个盘符，可通过移动到该路径下导出文件

```
jax777@DESKTOP-K49UI90:/mnt$ ls
c  d
jax777@DESKTOP-K49UI90:/mnt$ pwd
/mnt
jax777@DESKTOP-K49UI90:/mnt$ ls
c  d
jax777@DESKTOP-K49UI90:/mnt$ cd c
jax777@DESKTOP-K49UI90:/mnt/c$ ls
ls: cannot read symbolic link 'Documents and Settings': Permission denied
ls: cannot access 'hiberfil.sys': Permission denied
ls: cannot access 'pagefile.sys': Permission denied
ls: cannot access 'swapfile.sys': Permission denied
'$360Section'                       AppData                   Perl64                       Temp           pagefile.sys
'$Recycle.Bin'                      BOOTNXT                  'Program Files'               Users          phpStudy
 360Downloads                       CloudMusic               'Program Files (x86)'         Windows        qycache
 360SANDBOX                        'Documents and Settings'   ProgramData                  WpdPack        swapfile.sys
```

- 在win10 中寻找wsl 中的文件

  `C:\Users\jax777\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState\rootfs\home` 路径下

  `CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc` 每个子系统不同 路径不同

  当然可以在wsl 中创建一个 `tjfhvgbjykvghjbytkgvhj` 随机字符的文件，然后在`C:\Users\jax777\AppData\Local\Packages` 搜索即可找到位置
