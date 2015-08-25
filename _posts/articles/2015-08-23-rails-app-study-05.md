---
layout: post
title:  Rails 应用开发笔记（五）
excerpt: 在应用中使用 Bootstrap
comments: true
share: true
categories: articles
---

Bootstrap 是最受欢迎的 HTML、CSS 和 JS 框架，用于开发响应式布局、移动设备优先的 WEB 项目，这对
前端开发的菜鸟们来说简直是个神器啊，你不用特别精通 HTML、CSS 和 JS 就可以开发出漂亮美观的网站，而且
除去了适配和兼容性的问题。现在就用它去拯救一下我丑陋的应用！

Bootstrap 是基于 Less 构建的，还提供了一套官方支持的 Sass 移植版，但是即将发布的 Bootstrap 4
已经全面迁移到 Sass 了，所以在本项目中采用了 Bootstrap 的 Sass 移植版。

> [Bootstrap for SASS](https://github.com/twbs/bootstrap-sass)

#### 安装

编辑 Gemfile，添加 bootstrap gem 包：

`gem 'bootstrap-sass', '~> 3.3.5'`

> 如果安装过程出错，可以参考一下我在 [stackoverflow](http://stackoverflow.com/questions/32168612/an-error-occurred-while-installing-autoprefixer-rails-5-2-1-2) 的提问。

运行 `bundle install` 后重启服务器

由于在安装 Bootstrap 之前已经生成了项目，所以需要把 .css 文件扩展改为 .scss

`$ mv app/assets/stylesheets/application.css app/assets/stylesheets/application.scss`

导入 Bootstrap 样式到 `app/assets/stylesheets/application.scss`：

{% highlight js %}
// "bootstrap-sprockets" must be imported before "bootstrap" and "bootstrap/variables"
@import "bootstrap-sprockets";
@import "bootstrap";
{% endhighlight %}

去掉 `application.scss` 中所有的 `//= require` 和 `//= require_tree` 声明，否则会影响 Bootstrap 的正常使用

引入 bootstrap js 到 `app/assets/javascripts/application.js`:

{% highlight js %}
//= require jquery
//= require bootstrap-sprockets
{% endhighlight %}

#### 页面布局

页面布局可以通过 Bootstrap 的栅格系统轻松实现，Bootstrap提供了一套响应式、移动设备优先的流式栅格系统，随着屏幕或视口（viewport）尺寸的增加，系统会自动分为最多12列。

> http://v3.bootcss.com/css/#grid

Bootstrap 的栅格系统可以在多种屏幕设备上工作，主要依赖于4中预定义类：

1. col-xs-*: 超小屏幕 手机 (<768px)
2. col-sm-*: 小屏幕 平板 (≥768px)
3. col-md-*: 中等屏幕 桌面显示器 (≥992px)
4. col-lg-*: 大屏幕 大桌面显示器 (≥1200px)

以本应用为例：

{% highlight html %}
<div id="main-container" class="container">
  <div class="row">

    <div class="col-xs-1 col-sm-2 col-md-3">
      <h3>第一部分</h3>
    </div>
<!--     在中等屏幕下，第一部分占3列
    在小屏幕下，第一部分占2列
    在超小屏幕下，第一部分占1列 -->

    <div class="col-xs-11 col-sm-10 col-md-9">
      <h3>第二部分</h3>
      <%= yield %>
    </div>
<!--     在中等屏幕下，第一部分占9列
    在小屏幕下，第一部分占10列
    在超小屏幕下，第一部分占11列 -->

  </div>
</div>
{% endhighlight %}

<figure>
    <img src="/images/20150823-01.png">
    <img src="/images/20150823-02.png">
</figure>

通过这种方式，应用可以适应不同的屏幕尺寸。

#### 导航栏

{% highlight html %}
<nav class="navbar navbar-inverse" role="navigation">
  <div class="container">
    <div class="navbar-header">
      <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
      </button>
      <a class="navbar-brand">Rails MongoDB</a>
    </div>
    <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
      <ul class="nav navbar-nav">
        <li class="active"><%= link_to "首页", home_path %></li>
      </ul>
        <% if current_user %>
          <ul class="nav navbar-nav navbar-right">
            <li class="dropdown">
              <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false"><%= current_user.user_name %><span class="caret"></span></a>
              <ul class="dropdown-menu">
                <li><%= link_to "我的账户", edit_user_path(current_user) %></li>
                <li><%= link_to "我的文章", user_articles_path(current_user) %></li>
                <li><%= link_to "退出", log_out_path %></li>
              </ul>
            </li>
          </ul>
        <% else %>
        <ul class="nav navbar-nav navbar-right">
          <li><%= link_to "登录", log_in_path %></li>
          <li><%= link_to "注册", sign_up_path %></li>
        </ul>
        <% end %>
    </div>
  </div>
</nav>
{% endhighlight %}

<figure>
    <img src="/images/20150823-03.png">
</figure>

#### 表单

{% highlight html %}
<%= form_for(@user, html: {class: 'form-horizontal'}) do |f| %>
  <% if @user.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@user.errors.count, "error") %> prohibited this user from being saved:</h2>
      <ul>
      <% @user.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
      </ul>
    </div>
  <% end %>
  <div class="field form-group">
    <%= f.label :user_name, class: 'col-sm-2 control-label' %>
    <div class="col-sm-5 form-group-sm">
      <%= f.text_field :user_name, class: 'form-control' %>
    </div>
  </div>
  <div class="field form-group">
    <%= f.label :email, class: 'col-sm-2 control-label' %>
    <div class="col-sm-5">
      <%= f.text_field :email, class: 'form-control' %>
    </div>
  </div>
  <div class="field form-group">
    <%= f.label :password, class: 'col-sm-2 control-label' %>
    <div class="col-sm-5">
      <%= f.text_field :password, class: 'form-control' %>
    </div>
  </div>
  <div class="field form-group">
    <%= f.label :password_confirmation, class: 'col-sm-2 control-label' %>
    <div class="col-sm-5">
      <%= f.text_field :password_confirmation, class: 'form-control' %>
    </div>
  </div>
  <div class="actions form-group">
    <div class="col-sm-offset-2 col-sm-5">
      <%= f.submit nil, class: "btn btn-primary" %>
    </div>
  </div>
<% end %>
{% endhighlight %}

<figure>
    <img src="/images/20150823-06.png">
</figure>

#### 表格

{% highlight js %}
<table class="table">
  <tbody>
    <% @articles.each do |article| %>
      <tr>
        <td><h3><%= link_to article.title, user_article_path(@user, article) %></h3></td>
        <td><h3><%= link_to "删除", [@user, article], method: :delete, data: { confirm: 'Are you sure?' } %></h3></td>
      </tr>
    <% end %>
  </tbody>
</table>
{% endhighlight %}

<figure>
    <img src="/images/20150823-04.png">
</figure>

#### 分页

分页功能之前采用了 gem 包 kaminari，它同样支持 Bootstrap，它支持的主题有：

`bootstrap2, bootstrap3, bourbon, foundation, github, google, materialize, purecss, semantic_ui`

我们使用主题 bootstrap3：

`rails generate kaminari:views bootstrap3`

它自动会为代码中有分页的地方添加样式

<figure>
    <img src="/images/20150823-05.png">
</figure>

#### 搜索框

{% highlight js %}
<%= form_for home_path, method: :get, html: { role: "search", class: "navbar-form navbar-left"} do %>

  <div class="form-group">
     <%= text_field_tag :search, params[:search], class: "form-control" %>
  </div>

  <div class="form-group">
    <%= submit_tag "Search", class: "btn btn-default" %>
  </div>
<% end %>
{% endhighlight %}

 <figure>
    <img src="/images/20150823-07.png">
</figure>
