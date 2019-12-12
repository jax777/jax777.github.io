---
layout: post
title: python  ocr  简单验证码识别
categories: python
tag: 验证码
---


# 验证码
现在越来越多的登录点都有验证码了，没个验证码识别怎么好意思出去跟别人聊天！！！！

# python ocr
*Optical Character Recognition，光学字符识别）*是指电子设备（例如扫描仪或数码相机）检查纸上打印的字符，通过检测暗、亮的模式确定其形状，然后用字符识别方法将形状翻译成计算机文字的过程。
用ocr可以解决一部分简单的二维码是识别问题

# [pytesseract](https://github.com/madmaze/pytesseract "pytesseract")
Python-tesseract is an optical character recognition (OCR) tool for python. That is, it will recognize and "read" the text embedded in images.
可以使用pytesseract 实现简易的验证码识别

- 安装依赖
依赖PIL（Python Imaging Library）
ubuntu 下 安装
`pip install python-imaging`
`pip install pytesseract`


依赖[Google Tesseract OCR](https://github.com/tesseract-ocr/tesseract "Google Tesseract OCR")

[Google Tesseract OCR 安装wiki](https://github.com/tesseract-ocr/tesseract/wiki "Google Tesseract OCR 安装wiki")
ubuntu 下 安装

`apt-get install tesseract`
or 
`apt-get install tesseract-ocr (ubuntu 16.04)`


- 使用代码

如下 ,输入图片
```python
try:
    import Image
except ImportError:
    from PIL import Image
import pytesseract


print(pytesseract.image_to_string(Image.open('test.png')))
```

# 如何使用requests 下载验证码图片

参考[how-to-download-image-using-requests](https://stackoverflow.com/questions/13137817/how-to-download-image-using-requests "how-to-download-image-using-requests")

```python
import requests
import shutil

r = requests.get(url, stream=True)
if r.status_code == 200:
    with open(path, 'wb') as f:
        r.raw.decode_content = True
        shutil.copyfileobj(r.raw, f)
```

# tips
**注意请求验证码的时候，要把cookie带上，与登录请求保持一致，不然得到的也不是当前需要输入的验证码**