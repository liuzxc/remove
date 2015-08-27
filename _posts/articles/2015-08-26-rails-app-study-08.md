---
layout: post
title:  Rails 应用开发笔记（八）
excerpt: 使用 gravatar 生成用户头像
comments: true
share: true
categories: articles
---

无论是社交网站还是论坛，个性鲜明的头像可以很好的彰显你的个性和增加辨识度，但是对于开发简单的应用而言，
这确实是一件很麻烦的事情，保存用户的头像需要占用磁盘空间，也要想办法去控制图片的大小尺寸和分辨率，想想都
据的头疼，然而 [gravatar](http://en.gravatar.com/) 却为我们提供了一个很好的方案，你只需要提供给 email 地址，它会自动为你生成头像，并且提供个性化定制和尺寸控制，关键是使用起来非常简单，大量的网站采用了 gravatar。

gravatar 提供了多种语言的代码实例，以下是ruby版本的：

{% highlight ruby %}
# include MD5 gem, should be part of standard ruby install
require 'digest/md5'

# get the email from URL-parameters or what have you and make lowercase
email_address = params[:email].downcase

# create the md5 hash
hash = Digest::MD5.hexdigest(email_address)

# compile URL which can be used in <img src="RIGHT_HERE"...
image_src = "http://www.gravatar.com/avatar/#{hash}"
{% endhighlight %}

> [Code Samples/Reference Implementations](http://en.gravatar.com/site/implement/)

由于我们需要在视图的多个地方使用到 avatar，所以做好的办法是将其实现封装成一个helper方法，另外我们
需要有个字段来保存 avatar url，而不用每次都计算：

{% highlight ruby %}
#app/models/user.rb
class User
  ...
  field :avatar_url, type: String
  ...
end

#app/helpers/application_helper.rb
#添加size参数的作用是为了方便控制图片的大小
def avatar_url(user, size = 200)
  if user.avatar_url.nil?
    hash = Digest::MD5::hexdigest(user.email.downcase)
    user.avatar_url = "http://www.gravatar.com/avatar/#{hash}"
    user.save
  end
  user.avatar_url + "?s=#{size}"
end
{% endhighlight %}

要在导航栏和首页文章标题的开头来显示用户的头像，我们需要使用 image_tag 标签，：

{% highlight ruby %}
#app/views/layouts/application.html.erb
...
<ul class="nav navbar-nav navbar-right">
  <li class="dropdown">
    ...
    <%= image_tag avatar_url(current_user, 20) %>
    <ul class="dropdown-menu">
...

#app/views/articles/home.html.erb
...
<% @all_articles.each do |article| %>
  <li class="list-group-item">
    <h4>
    <%= image_tag avatar_url(article.user, 30) %>
    <%= link_to article.title, user_article_path(article.user, article) %>
  </h4>
  </li>
<% end %>
...
{% endhighlight %}

刷新页面就可以看到头像了：

<figure>
    <img src="/images/20150826-01.png">
</figure>