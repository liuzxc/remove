---
layout: post
title: linux 常用命令
excerpt: linux
comments: true
categories: blog
share: true
---

####ps 产看当前进程状态

* ps - :显示所有进程，等同于-e
* ps -u uid or username :选择有效的用户id或者是用户名

####chown 更改指定文件的拥有者群组

> chown [选项]... [所有者][:[组]] 文件...

{% highlight sh %}
chown user:group log.txt //更改文件拥有者和群组
chown user log.txt    //更改文件拥有者
chown :group log.txt //只更改文件群组
// -R: 改变指定目录以及其子目录下的所有文件的拥有者和群组
// -v: 显示详细信息
chown -R -v user:group log
{% endhighlight %}

####find 寻找指定目录下的文件

{% highlight sh %}
find . -name "*.pyc" -exec rm -rf {} \;
{% endhighlight %}
