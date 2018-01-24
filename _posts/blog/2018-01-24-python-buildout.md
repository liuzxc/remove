---
layout: post
title: Python - What's PyPi, pip, easy_install, setuptools, eggs, wheel, Buildout?
excerpt: 恶心的 Python 包管理
categories: blog
comments: true
share: true
---

在开始之前，可以先看看下面这篇有意思的博客，可以帮助你更好的了解 Python 的包管理：

[Python Packaging 编年史](http://wing2south.com/post/python-packaging-timeline/)

## What is PyPI?

> The Python Package Index (aka PyPI) is the official third-party software repository for the Python

PyPI(Python Package Index) 是 python 官方的第三方库的仓库，所有人都可以下载第三方库或上传自己开发的库到 PyPI。PyPI 推荐使用 pip 包管理器来下载第三方库。


## What is pip?

> pip (package manager). pip is a is a command-line program used to {install, remove, update, …} software packages written in Python.

1. [https://pip.pypa.io/en/stable/](https://pip.pypa.io/en/stable/)

2. [https://pip.pypa.io/en/stable/installing/](https://pip.pypa.io/en/stable/installing/)


## What is easy_install?

> Easy Install is a python module (``easy_install``) bundled with ``setuptools``
that lets you automatically download, build, install, and manage Python
packages.

可见 easy_install 只是 setuptools 的一个 module，它拥有部分的包管理的功能，但是没有 pip 那么全。

[esay_install 文档](https://svn.python.org/projects/sandbox/branches/setuptools-0.6/EasyInstall.txt)

## pip vs easy_install

[pip 与 easy_install 的对比](https://pip.readthedocs.io/en/1.1/other-tools.html#pip-compared-to-easy-install)

简单来说，pip 比 easy_install 出现的要晚，pip 的出现是为了改进 easy_install，更好的支持 python 包管理，但是 pip 也不能完全替代 easy_install，有些功能它依然不如 easy_install，比如 pip 不兼容一些广泛使用而且高度定制化的包，比如 distutils 和 setuptools。

## What is distutils?

> The distutils package provides support for building and installing additional modules into a Python installation

distutils 是 python  标准库的一部分，用来在 python 环境中构建和安装额外的包。

distutils2 是 distutils 的高级版本，不过已经夭折，所以不需要关心了。


## What is setuptools？

> Easily download, build, install, upgrade, and uninstall Python packages.

Setuptools is a collection of enhancements to the Python distutils that allow developers to more easily build and distribute Python packages, especially ones that have dependencies on other packages

简单说 setuptools 是 distutils 的增强版

[setuptools 介绍](https://setuptools.readthedocs.io/en/latest/setuptools.html)

[setuptools 安装](https://packaging.python.org/tutorials/installing-packages/)


## what is ez_setup.py?

ez_setup.py 是一个引导文件，可以帮助你自动的下载和安装 setuptools.

[source code](https://bootstrap.pypa.io/ez_setup.py)

## What is Eggs?

> "Eggs are to Pythons as Jars are to Java…”

Eggs 格式是 setuptools 引入的一种文件格式，它使用 .egg 扩展名，用于 Python 模块的安装，setuptools 可以识别这种格式，并解析它，安装它。

[Eggs 介绍](http://peak.telecommunity.com/DevCenter/PythonEggs)


## What is Wheel?

> Wheels are the new standard of python distribution and are intended to replace eggs.

wheel 本质上是一个 zip 包格式，它使用 .whl 扩展名，用于 python 模块的安装，它的出现是为了替代 Eggs。Wheel 要优于 Eggs， 因为 Eggs 诞生的比较早，不符合目前的一些 package 的相关标准，而 wheels 并不存在这个问题。

1. [http://wheel.readthedocs.io/en/stable/](http://wheel.readthedocs.io/en/stable/)
2. [https://pythonwheels.com/](https://pythonwheels.com/)


## what is Buildout?

> Buildout is a tool for automating software assembly.

[Buildout](http://docs.buildout.org/en/latest/index.html) 是一个基于 Python 的构建工具, 通过一个配置文件，可以从多个部分创建、组装并部署你的应用，即使应用包含了非 Python 的组件，Buildout也能够胜任。Buildout 不但能够像 setuptools 一样自动更新或下载安装依赖包，而且还能够像 virtualenv 一样，构建一个封闭隔离的开发环境。(引用自博客  [用Buildout来构建Python项目](https://lxneng.com/posts/192))

Buildout 这个项目主要是为了解决两个问题：

1. Application-centric assembly and deployment
2. Repeatable assembly of programs from Python software distributions

[zc.buildout 的介绍](https://pypi.python.org/pypi/zc.buildout)

Buildout 的安装通常由一个引导文件来完成:

下载链接：[https://github.com/buildout/buildout/blob/master/bootstrap/bootstrap.py][1]

同时还需要一个 buildout.cfg 的配置文件。


[1]: https://github.com/buildout/buildout/blob/master/bootstrap/bootstrap.py
