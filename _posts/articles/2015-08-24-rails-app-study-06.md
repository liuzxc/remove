---
layout: post
title:  Rails 应用开发笔记（六）
excerpt: 实现简单的搜索功能
comments: true
share: true
categories: articles
---

一般的网站都会有搜索功能，本篇将实现一个简单的搜索功能，搜索匹配的文章标题和内容。

#### 创建搜索栏

搜索功能相对来说是一个比较独立的功能，不适合把它放在已有的视图文件中，随着需求的变化，有可能会添加
新的搜索需求，所以我们在views目录单独创建一个 search 文件夹，将搜索框的代码单独放在其中：


{% highlight erb %}
＃app/views/search/_search.html.erb
<%= form_for home_path, method: :get, html: { role: "search", class: "navbar-form navbar-left"} do %>
  <div class="form-group">
     <%= text_field_tag :search, params[:search], class: "form-control" %>
  </div>

  <div class="form-group">
    <%= submit_tag "Search", class: "btn btn-default" %>
  </div>
<% end %>
{% endhighlight %}

然后我们将搜索框放入导航栏中：


{% highlight erb %}
#app/views/layouts/application.html.erb
<div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
  <ul class="nav navbar-nav">
    <li class="active"><%= link_to "首页", home_path %></li>
  </ul>

  <%= render 'search/search' %>

  <% if current_user %>
  .......
</div>
{% endhighlight %}

<figure>
    <img src="/images/20150824-01.png">
</figure>

#### 实现搜索功能

服务器在收到浏览器发来的搜索请求之后，需要有一个搜索的方法来处理该请求，然后将搜索结果返回给浏览器，
由于该搜索功能主要是搜文章的标题和内容，所以需要 Article 控制器来处理该请求：

{% highlight ruby %}
#app/controller/articles_controller.rb
def home
  if params[:search]
    @all_articles = Article.any_of({title: /#{params[:search]}/i}, {content: /#{params[:search]}/i})
                           .page params[:page]
  else
    @all_articles = Article.all.page params[:page]
  end
end
{% endhighlight %}

home 方法本来是用于在主页显示文章的，由于搜索后的文章也要显示给用户，所以此处改造了这个方法，如果有
params[:search] 参数，表示显示搜索到的文章，否则显示所有文章。此处的代码看起来没有什么问题，可以正常
工作，但是 Rails 是标准的 MVC 架构，控制器的主要作用是根据客户端的请求和参数选择合适的模型去处理数据，真正
和数据库交互的工作应该交由模型层去实现，而且控制层的代码不便于测试，所以此处我们要将查询功能放入 Article
model 中：

{% highlight ruby %}
#app/models/article.rb
class Article
  ....
  def self.search(param)
    any_of({title: /#{param}/i}, {content: /#{param}/i})
  end
  ...
end

#app/controller/articles_controller.rb
def home
  if params[:search]
    @all_articles = Article.search(params[:search]).page params[:page]
  else
    @all_articles = Article.all.page params[:page]
  end
end
{% endhighlight %}

这样一来，一个简单的搜索功能就实现了

<figure>
    <img src="/images/20150824-02.png">
</figure>
