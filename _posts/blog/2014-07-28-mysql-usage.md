---
layout: post
title: Learn Mysql
excerpt:
comments: true
categories: blog
share: true
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

####mysql建立索引的几大原则

引用自：[美团技术博客](http://tech.meituan.com/mysql-index.html)

1. 最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

2. =和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式

3. 尽量选择区分度高的列作为索引,区分度的公式是count(distinct col)/count(*)，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录

4. 索引列不能参与计算，保持列“干净”，比如from_unixtime(create_time) = ’2014-05-29’就不能使用到索引，原因很简单，b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成create_time = unix_timestamp(’2014-05-29’);

5. 尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可