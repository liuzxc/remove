---
layout: post
title: Learn Mysql
excerpt: mysql 常见操作
comments: true
categories: blog
share: true
---

#启动和关闭mysql服务

##linux：
`$ /etc/init.d/mysql start|stop|restart|status`

##mac：
`$ /Library/StartupItems/MySQLCOM/MySQLCOM start|stop|restart`

#备份和还原数据库

##备份：
`$ mysqldump -u (user_name) -p (db_name) (table_name) > 备份文件名称`

* 没有table_name时备份整个数据库；
* 备份文件名可加上绝对路径；

##还原：
`$ mysql -u root -p (db_name) < 备份文件名`

#mysql常用命令行参数
* -h : 连接的服务器名或者IP
* -u : 用户名
* -p : 密码
* -D : 使用哪个数据库
* -e : 执行sql语句
* --reconnect : 如果与服务器之间的连接断开，自动尝试重新连接
* --max_allowed_packet : 服务器接收／发送包的最大长度

#### 数据和表

##### 创建数据库和表

1. create database gregs_list CHARACTER set utf8mb4 COLLATE = utf8mb4_unicode_ci; 创建数据库 gregs_list
2. show databases; 列出当前所有的数据库
3. use gregs_list; 使用 gregs_list 这个数据库
4. create table my_contacts
    (
    last_name varchar(30),
    first_name varchar(20),
    email varchar(50),
    gender char(1),
    birthday date,
    profession varchar(50),
    location varchar(50),
    status varchar(20),
    interests varchar(100),
    seeking varchar(100)
    ); 创建 my_contacts 表
5. desc my_contacts; 查看 my_contacts 表
6. drop table my_contacts; 删除 my_contacts 表
7. show create table my_contacts; 返回建表的 create table 语句

##### insert 语句

`insert into your_table (column_name1, column_name2, ...) values ('value1', 'value2', ...);`

列名可省略：

`insert into your_table values ('value1', 'value2', ...);`

insert into ... select:

{% highlight sql %}
INSERT INTO table (col1, col2, ..., coln)
SELECT col1, col2, ..., coln
FROM table
WHERE entry_date < '2011-01-01 00:00:00';
{% endhighlight %}

##### select 语句

* Like与通配符
	1. % : 表示任意数量的未知字符串的替身；
	2. _ : 一个未知字符串的替身；

##### delete 语句

`delete from your_table where ....`

##### update 语句

`update your_table set first_column = 'new_value', second_column = 'new_value1' where ...;`

使用 case 表达式 update

{% highlight sql %}
update my_table
set new_column =
case
  when column1 = somavalue1
  when column2 = somevalue2
  else newvalue3
end;
{% endhighlight %}

##### alter 语句

1. change 可同时改变现有列的名称和数据类型
2. modify 修改现有列的数据类型和位置
3. add    在当前表中添加一列
4. drop   从表中删除某列
5. rename to 修改表名
6. add(drop) index (COLUMN_NAME) 添加(删除)索引

{% highlight sql %}
alter table my_contacts
add column contact_id INT NOT NULL AUTO_INCREMENT FIRST,
add primary key (contact_id);

ALTER TABLE TABLE_NAME ADD INDEX (COLUMN_NAME);
{% endhighlight %}

> 可以使用first, last, before column_name, after column_name, second, third 等关键字调整列的顺序

####mysql建立索引的几大原则

引用自：[美团技术博客](http://tech.meituan.com/mysql-index.html)

1. 最左前缀匹配原则，非常重要的原则，mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

2. =和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式

3. 尽量选择区分度高的列作为索引,区分度的公式是count(distinct col)/count(*)，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录

4. 索引列不能参与计算，保持列“干净”，比如from_unixtime(create_time) = ’2014-05-29’就不能使用到索引，原因很简单，b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成create_time = unix_timestamp(’2014-05-29’);

5. 尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可