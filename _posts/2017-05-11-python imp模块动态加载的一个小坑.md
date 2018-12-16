---
layout: post
title: python imp模块动态加载的一个小坑
---

# 前言
有个动态添加插件的需求，将所有插件存在一个目录下，主程序运行时动态加载目录里的插件。**但是第一次写的过程中，发现一个插件都会被加载两次，这就有了这篇文章。**


------------
# tips
于是去网上找了imp的用法，简单写了一下。（注意插件目录下不能有__init__.py文件，按照下面的写法也会加载这个空文件）
```python
def load_plugins(self):
    p_name = os.listdir('lib/plugins')
    for name in p_name:
        fp, pathname, description = imp.find_module(os.path.splitext(name)[0], ['lib/plugins'])
        m = imp.load_module(os.path.splitext(name)[0], fp, pathname, description)
        self.http_plugins.append(m)



```
经过检查发现，python第一次加载插件之后会在目录下生成pyc文件（由py文件经过编译后二进制文件，可提高加载速度）。然而第二次运行主程序时，plugins目录下就会多出相同的pyc文件，同时pyc文件也是可以被imp模块添加的这就导致了同样的插件在第二次运行时被加载两次。


重写一下，加个去重，以文件名为键（不含后缀），利用字典保证只添加一次。
```python
def load_plugins(self):
    p_name = os.listdir('lib/plugins')
    detect_names = {}

    for name in p_name:
        main_name = name.split('.')[0]  # file name
        if main_name in detect_names:
            detect_names[main_name] = 1
        else:
            detect_names[main_name] = 0
    for i in detect_names:
            fp, pathname, description = imp.find_module(i, ['lib/plugins'])
            m = imp.load_module(i, fp, pathname, description)
            self.normal_plugins.append(m)
```


# 好了结束了，现在可以愉快的去玩耍了。