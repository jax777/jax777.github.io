---
layout: post
title: browser_xss_fuzzer依靠浏览器的xss自动化检测实践
categories: 前端
tag: xss
---

# 基本思想
完全依靠前端代码实现xss的fuzz，通过浏览器打开一个html实现在各个浏览器环境下的fuzz。需要针对不同浏览器测试时，只需在各个浏览器中打开测试页面即可。
主页面与测试页面交互依靠postMessage传递信息，做出判断。

![](/styles/images/2018-12-16/demo.png)
### 基本设计流程如下

![](/styles/images/2018-12-16/procedure.png)



# 主要功能块
- 提供输入参数 url、method、post
- 建立xss fuzz的payload集合（这里的payload需要使得漏洞页面执行`window.parent.postMessage(window.location.href,'*');` 例如这种`<img src=x onerror=window.parent.postMessage(window.location.href,'*')>`）
- 建立隐藏iframe页面（已将参数替换为payload的测试页面），依靠浏览器自身对测试页面进行解析
- 接收message 做出xss存在性判断
- 显示存在漏洞的请求


# tips
## post请求的实现
由于browser_xss_fuzzer测试需要依靠浏览器解析，对于post请求不宜用ajax实现，这里采用在空iframe中建立隐藏form表单的形式实现，通过表单自动提交，完成post请求的测试。
具体表单建立代码如下
```javascript
function createInput(sfForm,type,name,value) 
    { 
      var tmpInput = document.createElement("input"); 
      tmpInput.type = type; 
      tmpInput.name = name; 
      tmpInput.value = value; 
      sfForm.appendChild(tmpInput); 
    } 
```

# 实现情况
给大家看一下实现的界面
![](/styles/images/2018-12-16/ui.png)