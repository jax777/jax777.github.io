---
layout: post
title: nginx配置的一些 tips
categories: server
tag: tips
---

## nginx .htaccess 文件规则配置 基本和 Apache兼容

- `<IfDefine>` 指令

    封装一组只有在启动时当测试结果为真时才生效的指令
    作用域 server config, virtual host, directory, .htaccess

- `<IFModule mod_rewrite.c> </IFModule>` 指令

    封装指令并根据指定的模块是否启用为条件而决定是否进行处理
    作用域 server config, virtual host, directory, .htaccess