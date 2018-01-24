---
layout: post
title: Rails 查询数据库的几种方式
excerpt: Active Record, mysql2
categories: blog
comments: true
share: true
---

其实，了解rails查询数据库最好的地方是[rails guide](http://guides.rubyonrails.org/active_record_querying.html)

#### 从数据库检索对象

Active Record提供了很多方法供你调用
例如:
{% highlight ruby %}
User.find()
User.where()
User.select()
......
{% endhighlight %}

以上方法会返回一个包含user对象的的数组。

这种操作的优点有：

* 使用简便，不需要写原生的sql，就算是不会写sql也可以轻松掌握；
* 由于返回的记录是对象，你可以通过该对象操作对应记录的任意字段；

同时，这种写法也存在一些缺点：

* 性能不是很好，速度慢，因为返回的是整个对象，所以很耗内存；
* 不适合用于复杂的查询，复杂的查询会让查询语句很长，无论是从性能还是外观看起来都不是很好；

#### 使用`find_by_sql`

该方法支持你写原生的sql语句

{% highlight ruby %}
User.find_by_sql("select* from users where .....")
{% endhighlight %}

以上方法返回的也是一个包含对象的数组。如果你的查询条件很复杂，写原生sql是一个好的选择。

以上两种方法返回的都是包含对象的数组，很显然，针对大量数据的查询，这些方法很不实用，速度很慢而且很耗内存。
有些时候，我们只需要某些字段的值，而不是整个对象，强大的rails已经提供了更好的方法：

{% highlight ruby %}
User.connection.select_values("select user_name from users where ...")
{% endhighlight %}

如果我只需要得到user表的user_name, 该方法会返回一个包含user_name的数组

{% highlight ruby %}
User.connection.select_all("select user_name from users where ...")
{% endhighlight %}

该方法会返回一个元素是hash的数组(like: `[{user_name: "jason"}, {user_name: "jack"}]`)
很明显，这两种方法的效率会更高，更节省内存。


#### `ActiveRecord::Base.connection.execute`

{% highlight ruby %}
rows = ActiveRecord::Base.connection.execute('select * from users')
rows.each { |row| puts row }
{% endhighlight %}

result 是一个`Mysql2::Result`的实例，row是包含一条user记录的数组。

#### mysql2

如果你只需要在ruby环境下连接mysql，你可以使用`mysql2`。

> Mysql2 - A modern, simple and very fast MySQL library for Ruby

{% highlight ruby %}
require 'mysql2'

client = Mysql2::Client.new(:host => HOST, :username => USERNAME, :database => DATABASE)
client.query("select * from users;")
{% endhighlight %}