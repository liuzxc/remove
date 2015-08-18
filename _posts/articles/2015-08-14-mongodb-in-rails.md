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

后台启动mongod，并将日志输出到相应的文件

`mongod --fork --logpath /var/log/mongodb.log`

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

#### 关联

文章一般都会有评论，因此创建一个 Comment 模型与 Article 模型相关联，mongoid 的模型关联与 ActiveRecord
有稍稍有些不同，但是用法是相似的，它使用 embeds_many 方法而不是 has_many 方法。ActiveRecord 是通过外键来
关联不同的数据库表，而 mongodb 没有外键的概念，`embeds_many :comments` 表示将 comments 这个文档嵌入到 Article 文档中。

{% highlight ruby %}
class Article
  include Mongoid::Document
  field :title, type: String
  field :content, type: String
  field :category, type: String, default: 'diary'
  validates :title, presence: true

  embeds_many :comments
end
{% endhighlight %}

创建一个 Comment 模型：

`$ rails g model comment name:string content:text

然后在 Comment 模型中定义它与 Article 的关联关系：

{% highlight ruby %}
class Comment
  include Mongoid::Document
  field :name
  field :content
  embedded_in :article, :inverse_of => :comments
end
{% endhighlight %}

> inverse_of 用来告诉 Rails 两个模型之间的关系,与 ActiveRecord 的用法是一致的
> 参考链接：http://guides.ruby-china.org/association_basics.html

然后修改 `routes.rb`

{% highlight ruby %}
resources :articles do
  resources :comments
end
{% endhighlight %}

创建 comments controller，并添加一个 create 方法

`$ rails g controller comments`

{% highlight ruby %}
CommentsController < ApplicationController
  def create
    @article = Article.find(params[:article_id])
    @comment = @article.comments.create!(params[:comment])
    redirect_to @article, :notice => "Comment created!"
  end
end
{% endhighlight %}

添加以下代码到 article 的 show view，用来显示 comments

{% highlight ruby %}
<% if @article.comments.size > 0 %>
  <h2>Comments</h2>
  <% for comment in @article.comments %>
    <h3><%= comment.name %></h3>
    <p><%= comment.content %></p>
  <% end %>
<% end %>

<h2>New Comment</h2>

<%= form_for [@article, Comment.new] do |f| %>
  <p><%= f.label :name %> <%= f.text_field :name %></p>
  <p><%= f.text_area :content, :rows => 10 %></p>
  <p><%= f.submit %></p>
<% end %>
{% endhighlight %}

<figure>
    <img src="/images/20150814-03.png">
</figure>