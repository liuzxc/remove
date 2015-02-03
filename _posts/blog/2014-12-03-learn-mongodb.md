---
layout: post
title: Learn Mongodb
excerpt:
categories: blog
comments: true
---

#### Installation

{% highlight bash %}
//Update Homebrew’s package database
brew update
//Install MongoDB
brew install mongodb
{% endhighlight %}

#### Run Mongodb

在你第一次启动mongodb之前，需要创建一个供mongodb写数据的文件夹，默认使用/data/db目录，
并且要确保这个目录有读写权限。

{% highlight bash %}
mkdir -p /data/db
chown -R $USER:$GRUOP /data/db
mongod
mongo //连接到数据库
{% endhighlight %}