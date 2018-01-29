---
layout: post
title: Learn Mongodb
excerpt: Mongodb
categories: blog
comments: true
share: true
---

#### Installation

```bash
//Update Homebrew’s package database
brew update
//Install MongoDB
brew install mongodb
```

#### Run Mongodb

在你第一次启动mongodb之前，需要创建一个供mongodb写数据的文件夹，默认使用/data/db目录，
并且要确保这个目录有读写权限。

```bash
mkdir -p /data/db
chown -R $USER:$GRUOP /data/db
mongod
mongo //连接到数据库
```
