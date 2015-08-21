---
layout: post
title:  Rails 应用开发笔记（三）
excerpt: 分页功能的实现
comments: true
share: true
categories: articles
---

只要是web应用的开发，分页的功能都是必不可少的，Rails 强大的社区生态环境诞生了丰富多样的 gem package，
大大提高了 web 应用的生产效率。有很多的 gem package 提供了分页的功能，今天要介绍的是其中最为强大的一种：
kaminari。

Kaminari 是一个强大的，可定制化的分页引擎，支持多种 ORM(ActiveRecord, mongoid)，多种 web 框架（Rails，
Sinatra，Grape)，以及多种模版引擎(ERB, Haml, Slim).

####安装

在 Gemfile 中添加以下代码:

`gem 'kaminari'`

然后运行 bundle 安装。

####使用

以用户的文章分页来试验如何通过kaminari来实现分页功能

1. 在model中配置分页数量, paginates_per表示每一页的文章数量

{% highlight ruby %}
  class Article
    ...
    paginates_per 5
  end
{% endhighlight %}

2. 在 Article的控制层中接收分页参数 params[:page]

{% highlight ruby %}
  def index
    @user = User.find(params[:user_id])
    @articles = @user.articles.page params[:page]
  end
{% endhighlight %}

3. 在视图层添加paginate helper方法

{% highlight ruby %}
  <%= paginate @articles %>
{% endhighlight %}

一个简单的分页功能就实现了，是不是超级简单呢？

<figure>
    <img src="/images/20150821-01.png">
</figure>