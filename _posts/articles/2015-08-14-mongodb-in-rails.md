---
layout: post
title: 简单介绍如何在 Rails 项目中使用 MongoDB
excerpt:
comments: true
share: true
categories: articles
---

做游戏开发快两年了，项目中使用的数据库一直都是 Mysql ＋ Redis，不过渐渐发现，无论是公司的新项目
还是 GitHub 上的一些开源项目，都开始在使用 MongoDB 了。虽然我还没有在项目中实际使用 MongoDB，
但是我已经有兴趣去尝试，暂时就先拿一个 rails demo 来试手！


#### 安装MongoBD

使用 brew 命令就可以安装 mongodb

`brew install mongodb`

安装完成以后需要创建一个 data 目录，进程 mongod 会把数据写在这个目录下面，需要注意的是要确保 mongod
对这个目录有足够的读写权限

`mkdir -p /data/db`

运行 mongodb

`mongod`

启动 mongo shell

`mongo`

#### mongoid VS ActiveRecord

mongoid 和 ActiveRecord 都是数据库对象关系映射（database ORM），准确的说，mongoid 是 ODM
（Object Document Mappers）。不同之处在于，ActiveRecord 针对关系型数据库，比如 Mysql，SQLServer，
PostgreSQL等，所以 ActiveRecord 不适用于 Nosql，所以如果 rails 应用要使用 mongodb，必须用 mongoid。

#### Rails 项目中使用 mongoid

> 参考链接：http://docs.mongodb.org/ecosystem/tutorial/ruby-mongoid-tutorial/#ruby-mongoid-tutorial

mongoid是一个gem包，只需要把它添加到 Gemfile 中：

gem 'mongoid', '~> 5.0.0.beta'

然后运行 bundle install

mongoid 的配置要通过 mongoid.yml，类似于 database.yml，可以通过以下命令自动创建：

`rails g mongoid:config`

接下来我们尝试搭建一个小小的博客应用，该应用可以对文章进行增删改查操作，文章有标题和内容属性，通过以下命令创建一个 Article model 及对应的 controller 和 view。

`rails g scaffold article title:string content:text`

 > 由于 mongodb 是 nosql 数据库，是 schema-less (无模式的），因此不需要 migration 操作

 然后启动 rails server，访问 http://localhost:3000/articles，即可进行文章的增删改查操作。

#### 添加新字段

如果我们现在想要给 Article 添加一个 category 的字段该怎么办呢？

{% highlight ruby %}
class Article
  include Mongoid::Document
  field :title,   type: String
  field :content, type: String
  field :category,  type: String, default: 'diary'
end
{% endhighlight %}

只需要在 Article model 中添加一个新的 field 就可以了，然后再修改其他的相应的文件。由此我们可以看出 mongodb
这种无模式数据库的优点所在了，即添加新的字段非常容易。相比传统的 mysql，添加新的字段是比较麻烦的，尤其是
对于那些数据量很大的表，添加新的字段会锁表，造成当前数据无法写入，影响用户体验。而 mongodb 可以很轻松的解决
这个问题。

<figure>
    <img src="/images/20150814-01.png">
</figure>

#### 数据验证

{% highlight ruby %}
class Article
  include Mongoid::Document
  field :title, type: String
  field :content, type: String
  field :category, type: String, default: 'diary'
  validates :title, presence: true
end
{% endhighlight %}

<figure>
    <img src="/images/20150814-02.png">
</figure>

mongoid 的数据验证和 ActiveRecord 的方法是一致的，因此可以通用。