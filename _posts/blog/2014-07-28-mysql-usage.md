---
layout: post
title: Learn Mysql
excerpt:
comments: true
categories: blog
---

#启动和关闭mysql服务

##linux：
$ /etc/init.d/mysql start|stop|restart|status

##mac：
$ /Library/StartupItems/MySQLCOM/MySQLCOM start|stop|restart

#备份和还原数据库

##备份：
$ mysqldump -u (user_name) -p (db_name) (table_name) > 备份文件名称

* 没有table_name时备份整个数据库；
* 备份文件名可加上绝对路径；

##还原：
$ mysql -u root -p (db_name) < 备份文件名

#mysql常用命令行参数
* -h : 连接的服务器名或者IP
* -u : 用户名
* -p : 密码
* -D : 使用哪个数据库
* -e : 执行sql语句
* --reconnect : 如果与服务器之间的连接断开，自动尝试重新连接
* --max_allowed_packet : 服务器接收／发送包的最大长度