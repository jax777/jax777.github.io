---
layout: post
title: sqlmap处理在 http response header 中的bool型注入
categories: java
tag: tips
---

## 说明

最近测试的时候发现一个登录口的注入点，是否成功登录或者用户名是否正确的判断存在于`response header`中的一个字段 `Message`，response content 始终为空。

请求包如下

```text
POST /xxx/login HTTP/1.1
Host: xxx.xxx.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:67.0) Gecko/20100101 Firefox/67.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 52
Connection: close
Cookie: JSESSIONID=862AFD6D496A7C6BB70EAEF7BF22FC7A

name=admin&password=xxxxx


response

HTTP/1.1 200 OK
Date: Thu, 30 May 2019 07:05:27 GMT
Server: Apache/2.4.29 (Ubuntu)
Content-Encoding: gzip
Content-Length: 0
Message:xxxxxxx
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

```

`response header`中的 `Message` 会依据name 用户名是否存在进行变化。

post的name参数存在sql注入，直接丢sqlmap测试仅能测出时间盲注，对 `response header` 的不同并没有做出判断。理论上应该存在bool型注入，sqlmap可能在页面相似判断的过程中出现了问题。

## 解决

一开始以为sqlmap试没有对 `response header` 的部分进行判别,搜索项目中 issues ，发现 5月6日 有人提了一个`Evaluation of boolean-based blind SQLi on HTTP response headers`

其中推荐使用 `--string`   `--regex` 来进行测试。

原本以为试 sqlmap 的页面相似检测没有涉及 `response header`，查阅源码发现sqlmap在做页面相似度判断时已经将**完整的http响应（保护 response header）写入**`rawResponse`留待下一步判断。sqlmap没有做出检测应该是页面变化太小，导致页面相似度判断没有检测出不同的结果，最终只要在参数中添加  `--string="Message:name存在时的值"` 即可完成正常判别。

`rawResponse = "%s%s" % (listToStrValue(_ for _ in headers.headers if not _.startswith("%s:" % URI_HTTP_HEADER)) if headers else "", page)`

添加 --string="Message: 1024"

> https://github.com/sqlmapproject/sqlmap/issues/3518

```text
sqlmap -hh

Detection:
    These options can be used to customize the detection phase

    --level=LEVEL       Level of tests to perform (1-5, default 1)
    --risk=RISK         Risk of tests to perform (1-3, default 1)
    --string=STRING     String to match when query is evaluated to True
    --not-string=NOT..  String to match when query is evaluated to False
    --regexp=REGEXP     Regexp to match when query is evaluated to True
    --code=CODE         HTTP code to match when query is evaluated to True
    --text-only         Compare pages based only on the textual content
    --titles            Compare pages based only on their titles
```

