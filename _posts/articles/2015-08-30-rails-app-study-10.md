---
layout: post
title:  Rails 应用开发笔记（十）
excerpt: 在 Rails 中使用 Ajax
comments: true
share: true
categories: articles
---

作为一名后端开发人员，我总是对前端技术心生畏惧，前端技术总要涉及到各种前端语言和框架，比如 HTML，
CSS，Javascript，Ajax，jQuery等等，光是 HTML 那些标签就让我头疼不已，更别说还要具备一定的审美
能力。但是当我自己着手开发一个web app 的时候，我不得不去克服心中的畏惧和反感，因为你没法容忍在点击
收藏或点赞标签的时候，整个页面都被刷新，因此我试着去学习前端方面的知识，看了一些教程，尤其是 Head First
系列的图书，简单而生动，让我获益匪浅，而 stackoverflow 更是个巨大的宝库，在我遇到困难的时候，热心的
网友们给予了我极大的帮助，短短几天时间，我就可以熟练的使用 jQuery 和 Ajax 实现一些简单的效果，这对我
是极大的鼓舞。

在介绍我在项目中使用 Ajax 之前，先来了解几个基本的概念：

1. Javascript
2. Coffeescript
3. Ajax
4. jQuery
5. Unobtrusive JavaScript
6. jquery-ujs

#### 什么是 Javascript ?

Javascript 是一种解释型的脚本语言，被广泛的应用于 web 开发，主要用于给 HTML 页面增加动态功能，现在几乎所有的
浏览器都内置有 Javascript 引擎用来解释 Javascript。

#### 什么是 Coffeescript ?

Coffeescript 可以说是 Javascript 的变种，它抛弃了 Javascript 晦涩的语法和概念，使用了类似与 ruby/python 的
语法， Coffeescript 会被编译成 Javascript。

#### 什么是 Ajax ?

AJAX 即 `Asynchronous Javascript And XML`（异步 JavaScript 和 XML ），是指一种创建交互式网页应用的网页开发技术。JavaScript 可以向服务器发起请求，并处理响应，而且还能更新网页中的部分内容，而不用从服务器获取完整的页面数据，这种强大的技术对 web 开发至关重要。

#### 什么是 jQuery ?

jQuery 是一个优秀的 Javascript 库，它可以方便的处理网页元素，实现动画效果，以及方便的提供 Ajax 交互。

#### 什么是 Unobtrusive JavaScript（UJS) ?

我都不知道咋翻译这个词，[Rails Guide](http://guides.ruby-china.org/working_with_javascript_in_rails.html) 称其为剥离式 JavaScript。UJS 不是一种语言，不是 JavaScript 的变种，也算不上一种技术，维基百科称其为一种
使用 JavaScript 的通用方法或最佳实践。就是将 JavaScript 和普通的 HTML 代码分离开来，即网页内容/结构和其表现的行为分离开来，使代码清晰可读，并带来更好的扩展性。这显然很符合 Rails 社区的精神，因此 Rails 团队极力推荐使用这种方式编写 CoffeeScript 和 JavaScript。


了解了这些概念之后

#### 什么是 jquery-ujs

jquery-ujs 是 一个 gem 包，是 Rails 中的一个支持jQuery的剥离式脚本适配器，它提供的功能有：

1. 为各种操作增加确认对话框
2. 让普通超链接支持非 GET 请求
3. 让表单和超链接以异步的 Ajax 方式来提交数据
4. 已经提交按钮会自动变成禁用表单提交，以防止重复点击造成多次提交

> [jquery-ujs](https://github.com/rails/jquery-ujs)

了解完这些概念之后，我已经快要吐血了😓！！！

在本应用里，哪些地方需要用到 Ajax 技术呢？

1. 收藏和点赞功能；
2. 评论功能；
3. ‘我的主页’中的部分内容的显示（我的文章，我的收藏等）；

在这些地方用到 Ajax 的原因是为了提高用户体验和网站性能，即只与服务器进行部分数据交互，刷新部分页面，而不需要刷新整个页面。本篇以收藏功能为例，逐步讲解如何使用Ajax完善上篇讲到的收藏功能。

在用户点击收藏或者取消收藏链接的时候，我们希望发起一个 Ajax 的请求，只需要在 link_to 中添加 `remote: true` 这个参数，点击链接的时候，发往控制器的就是 Ajax 请求，使用 Javascript 处理：

{% highlight erb %}
#app/views/favorites/_favorite_link.html.erb
<% if favorite = current_user.favorites.where(article_id: @article.id).first %>
  <%= link_to favorite_path(favorite), method: :delete, remote: true do %>
    <span class="glyphicon glyphicon-bookmark"></span>
  <% end %>
<% else %>
  <%= link_to favorites_path(article_id: @article.id), method: :post, remote: true do %>
    <span class="glyphicon glyphicon-bookmark favorite_color"></span>
  <% end %>
<% end %>
{% endhighlight %}

点击收藏链接后， 通过 rails log 我们可以看到：

{% highlight ruby %}
Started POST "/favorites?article_id=55dd87307b843a3780000007" for ::1 at 2015-08-31 23:04:39 +0800
Processing by FavoritesController#create as JS
  Parameters: {"article_id"=>"55dd87307b843a3780000007"}
...
{% endhighlight %}

POST 请求被 FavoritesController 的 create 方法作为 JS 来处理:

{% highlight ruby %}
class FavoritesController < ApplicationController
  def create
    @article = Article.find(params[:article_id])
    current_user.favorites.create(article_id: params[:article_id])
    render :favorite
  end

  def destroy
    favorite = Favorite.find(params[:id])
    @article = favorite.article
    favorite.destroy
    render :favorite
  end
end
{% endhighlight %}

create 方法执行完毕后，会寻找合适的模版来渲染，新建一个 `favorite.js.erb` 视图文件，编写在客户端执行的 JS 代码：

{% highlight erb %}
$("#favorite").html("<%= escape_javascript(render partial: 'favorite_link') %>");
{% endhighlight %}

这是一段 jQuery 代码，意思是获取第一个匹配 `id="favorite"` 的元素，设其元素的 html 内容为渲染的 favorite_link 页面。

{% highlight erb %}
#app/views/articles/show.html.erb
<div id="favorite" class="col-md-4 text-left">
  <%= render 'favorites/favorite_link' %>
</div>
{% endhighlight %}

jQuery 会根据 id 找到这段代码，并替换其中的内容。

这样一来，我们就通过 Ajax 技术实现了局部更新。
