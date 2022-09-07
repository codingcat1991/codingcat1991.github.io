---
title: Python生成公共包（SDK）
category: Python
tags:
  - Python
keywords: SDK
cover: 1590516004.jpg
top_img: 1590516004.jpg
date: 2021-12-28 20:28:17
description: 很多时候我们希望将开发的模块代码弄成公用的模块，方便各个开发人员使用，那应该怎么做呢？
---


很多时候我们希望将开发的模块代码弄成公用的模块，方便各个开发人员使用，那应该怎么做呢？

## 项目结构

使用如下项目演示

```
python-sdk
    │  README.md
    │  setup.py
    │
    └─Demo
            __init__.py
```

其中Demo中只有一个方法，写在\_\_init\_\_.py模块中，如下:

```python
def say_hello():
    print("hello!")

```

setup.py用于安装Demo包，代码如下：

```python
from setuptools import setup, find_packages
 
setup(
	name = "demo",
	version = "0.1",
	url = '',
	long_description = open('README.md').read(),
	packages = find_packages(),
)
```

各参数含义如下：

| 参数             | 解释                                                      |
| ---------------- | --------------------------------------------------------- |
| name             | 包的名字                                                  |
| version          | 版本号，对保持适当的依赖关系很重要                        |
| packages         | 需要包含的子包列表，用find_packages()查找                 |
| url              | 包的链接，通常为 Github 上的链接，或者是 readthedocs 链接 |
| long_description | 将说明文件设置为README.md                                 |

## 创建包

执行`python setup.py bdist_egg`即可将Demo打包的一个SKD包，如下：

```
C:\Users\xxx\Desktop\python-sdk>python setup.py bdist_egg
running bdist_egg
running egg_info
creating demo.egg-info
writing demo.egg-info\PKG-INFO
writing dependency_links to demo.egg-info\dependency_links.txt
writing top-level names to demo.egg-info\top_level.txt
writing manifest file 'demo.egg-info\SOURCES.txt'
reading manifest file 'demo.egg-info\SOURCES.txt'
writing manifest file 'demo.egg-info\SOURCES.txt'
installing library code to build\bdist.win-amd64\egg
running install_lib
running build_py
creating build
creating build\lib
creating build\lib\Demo
copying Demo\__init__.py -> build\lib\Demo
creating build\bdist.win-amd64
creating build\bdist.win-amd64\egg
creating build\bdist.win-amd64\egg\Demo
copying build\lib\Demo\__init__.py -> build\bdist.win-amd64\egg\Demo
byte-compiling build\bdist.win-amd64\egg\Demo\__init__.py to __init__.cpython-37.pyc
creating build\bdist.win-amd64\egg\EGG-INFO
copying demo.egg-info\PKG-INFO -> build\bdist.win-amd64\egg\EGG-INFO
copying demo.egg-info\SOURCES.txt -> build\bdist.win-amd64\egg\EGG-INFO
copying demo.egg-info\dependency_links.txt -> build\bdist.win-amd64\egg\EGG-INFO
copying demo.egg-info\top_level.txt -> build\bdist.win-amd64\egg\EGG-INFO
zip_safe flag not set; analyzing archive contents...
creating dist
creating 'dist\demo-0.1-py3.7.egg' and adding 'build\bdist.win-amd64\egg' to it
removing 'build\bdist.win-amd64\egg' (and everything under it)
```

执行后，python-sdk目录下多了build、dist、demo.egg-info三个文件夹，目录结构如下：

```
─python-sdk
    │  README.md
    │  setup.py
    │
    ├─build
    │  ├─bdist.win-amd64
    │  └─lib
    │      └─Demo
    │              __init__.py
    │
    ├─Demo
    │      __init__.py
    │
    ├─demo.egg-info
    │      dependency_links.txt
    │      PKG-INFO
    │      SOURCES.txt
    │      top_level.txt
    │
    └─dist
            demo-0.1-py3.7.egg
```

将python-sdk下面的文件都打包到一个压缩包里面，就可以发给其他人使用了

## 使用

将受到的压缩包解压，并进入到包含setup.py所在的目录，执行如下命令安装包：

```
python setup.py install
```

安装过程如下：

```
C:\Users\xxx\Desktop\python-sdk>python setup.py install
running install
running bdist_egg
running egg_info
writing demo.egg-info\PKG-INFO
writing dependency_links to demo.egg-info\dependency_links.txt
writing top-level names to demo.egg-info\top_level.txt
reading manifest file 'demo.egg-info\SOURCES.txt'
writing manifest file 'demo.egg-info\SOURCES.txt'
installing library code to build\bdist.win-amd64\egg
running install_lib
running build_py
creating build\bdist.win-amd64\egg
creating build\bdist.win-amd64\egg\Demo
copying build\lib\Demo\__init__.py -> build\bdist.win-amd64\egg\Demo
byte-compiling build\bdist.win-amd64\egg\Demo\__init__.py to __init__.cpython-37.pyc
creating build\bdist.win-amd64\egg\EGG-INFO
copying demo.egg-info\PKG-INFO -> build\bdist.win-amd64\egg\EGG-INFO
copying demo.egg-info\SOURCES.txt -> build\bdist.win-amd64\egg\EGG-INFO
copying demo.egg-info\dependency_links.txt -> build\bdist.win-amd64\egg\EGG-INFO
copying demo.egg-info\top_level.txt -> build\bdist.win-amd64\egg\EGG-INFO
zip_safe flag not set; analyzing archive contents...
creating 'dist\demo-0.1-py3.7.egg' and adding 'build\bdist.win-amd64\egg' to it
removing 'build\bdist.win-amd64\egg' (and everything under it)
Processing demo-0.1-py3.7.egg
Removing e:\app\python\python37\lib\site-packages\demo-0.1-py3.7.egg
Copying demo-0.1-py3.7.egg to e:\app\python\python37\lib\site-packages
demo 0.1 is already the active version in easy-install.pth

Installed e:\app\python\python37\lib\site-packages\demo-0.1-py3.7.egg
Processing dependencies for demo==0.1
Finished processing dependencies for demo==0.1

```

进入到Python解释器并执行包中的函数如下：

```
C:\Users\xxx\Desktop\python-sdk>python
Python 3.7.5 (tags/v3.7.5:5c02a39a0b, Oct 15 2019, 00:11:34) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import Demo
>>> Demo.say_hello()
hello!
>>>
```

卸载包使用`pip uninstall`命令即可：

```
C:\Users\xxx\Desktop\python-sdk>pip uninstall Demo
Found existing installation: Demo 0.1
Uninstalling demo-0.1:
  Would remove:
    e:\app\python\python37\lib\site-packages\demo-0.1-py3.7.egg
Proceed (y/n)? y
  Successfully uninstalled bscp-config-0.1
```



