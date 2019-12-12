---
layout: post
title: python-nmap库的坑
categories: python
tag: 多进程
---

## PortScannerAsync 异步调用

```python
nm = nmap.PortScannerAsync()
nm.scan(hosts=ip, arguments=nmap_arguments,callback = nmap_result_callback)
```

- 这里这个`nmap_result_callback`由于 `python-nmap` 是利用子进程实现的，普通的全局变量在回调函数中修改与主进程无法通信。

可使用multiprocessing 中的dict 做全局变量，完成进程间通信。

```python
from multiprocessing import Manager

manager = Manager()

nmap_result = manager.dict()

```
