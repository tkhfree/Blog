---
title: python包管理
tags:
  - Python
comments: true
toc: true
mathjax: true
abbrlink: 8b0a4074
date: 2020-05-14 11:50:42
urlname:
categories:
thumbnail:
---

在安装Python包的过程中，经常涉及到distutils、setuptools、distribute、setup.py、easy_install、easy_install和pip等等。



### **distutils**

distutils 是 python 标准库的一部分，这个库的目的是为开发者提供一种方便的打包方式， 同时为使用者提供方便的安装方式。



我们经常使用的setup.py就是基于distutils实现的，然后通过setup.py就可以进行打包或者安装了。



看一个简单的例子，找一个目录创建三个文件foo.py、bar.py和setup.py，其中setup.py的内容如下：

```python
from distutils.core import setupsetup( name='fooBar', version='1.0', author='Will', author_email='wilber@sh.com', url='http://www.cnblogs.com/wilber2013/', py_modules=['foo', 'bar'],)
```

然后，在该目录中运行 python setup.py sdist ，会得到以下输出，同时生成了一个"fooBar-1.0.zip"包。

使用者就可以解压缩这个包然后执行 python setup.py install进行安装，然后就可以使用foo、bar这两个模块了



### **setuptools 和 distribute**

**setuptools**是对 distutils 的增强，尤其是引入了包依赖管理。我们可以通过ez_setup.py来安装setuptools。

至于**distribute**，它是setuptools的一个分支版本。分支的原因是有一部分开发者认为 setuptools 开发太慢。但现在，distribute 又合并回了 setuptools 中，所以可以认为它们是同一个东西。

前面看到setup.py可以创建一个压缩包，而setuptools使用了一种新的文件格式（**.egg**），可以为Python包创建 egg文件。setuptools 可以识别.egg文件，并解析、安装它。



easy_install

当安装好setuptools/distribute之后，我们就可以直接使用easy_install这个工具了：

1.从PyPI上安装一个包：当使用 easy_install package 命令后，easy_install 可以自动从 PyPI 上下载相关的包，并完成安装，升级

2.下载一个包安装：通过 easy_install package.tgz 命令可以安装一个已经下载的包

3.安装egg文件：通过 easy_install package.egg 可以安装一个egg格式的文件

通过 easy_install --help 命令可以获取该命令相关的帮助提示



根据上面的分析，可以看到setuptools/distribute和easy_install之间的关系：

*setuptools/distribute 都扩展了 distutils，提供了更多的功能

*easy_install是基于setuptools/distribute的一个工具，方便了包的安装和省级

### **pip**

pip是目前最流行的Python包管理工具，它被当作easy_install的替代品，但是仍有大量的功能建立在setuptools之上。

easy_install 有很多不足：安装事务是非原子操作，只支持 svn，没有提供卸载命令， 安装一系列包时需要写脚本。pip 解决了以上问题，已经成为新的事实标准。

pip的使用非常简单，并支持从任意能够通过 VCS 或浏览器访问到的地址安装 Python 包：

***安装:** pip install SomePackage

***卸载:** pip uninstall SomePackage

使用pip

在大家使用Python中，推荐使用pip进行Python包管理，pip的安装和使用都比较方便。

**pip安装**

pip的安装有两种常用的方式：

1.下载get-pip.py文件，然后执行 python get-pip.py 进行安装（如果没有安装setuptools，那么get-pip.py会帮忙安装）

2.现在pip源码包，然后通过setup.py进行安装

pip常用命令

对于pip，最常用的肯定还是 pip --help ，通过帮助文档，就可以大概知道如何使用命令和参数。



