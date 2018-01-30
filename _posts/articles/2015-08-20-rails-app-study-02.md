---
layout: post
title:  Rails 应用开发笔记（二）
excerpt: 添加用户注册和登录功能
comments: true
share: true
categories: articles
---

用户注册和登录功能是web应用的最基本功能之一，Rails 为我们提供了很方便的一些方法去实现这个功能。
在实现注册和登录功能之前，我们要了解几个概念：

#### has_secure_password

处于安全角度的考虑，用户的密码不会以纯文本的形式存储在数据库中，`ActiveModel has_secure_password` 方法可以对你的密码进行加密操作，并提供相应的数据验证和密码校验功能，你不用自己再去写这些代码，提高了编码效率。如果
要使用 `has_secure_password`，只需要添加少量代码和配置：

1. `has_secure_password` 方法要求你的用户表中有 password_digest 属性，用于存储加密后的密码，所以
你需要在 User 模型中添加一个 password_digest 字段。

2. 需要使用 bcrypt 这个 gem package，只需要在 Gemfile 中取消相应的注释即可。

  ```ruby
  # Use ActiveModel has_secure_password
  gem 'bcrypt', '~> 3.1.7'
  ```

3. 在User模型中加入 `has_secure_password` 方法，如果使用的是非关系型数据库，还需要 `include ActiveModel::SecurePassword` 模块。

```ruby
class User
  include Mongoid::Document
  include ActiveModel::SecurePassword
  .....
  has_secure_password
end
```

> 参考链接： http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password

#### 用户注册

用户注册即新用户的创建，只需要修改创建新用户的表单，添加 password 和 password_confirmation field.

```ruby
# view/users/_form.html.erb
<div class="field">
  <%= f.label :password %><br>
  <%= f.password_field :password %>
</div>

  <div class="field">
  <%= f.label :password_confirmation %><br>
  <%= f.password_field :password_confirmation %>
</div>
```

添加新的路由：

`get 'sign_up' => 'users#new'`

<figure>
    <img src="/images/20150820-01.png">
</figure>

#### 用户登录和退出

现在需要一个 session controller 去实现登录和退出的功能。

`rails g controller sessions`

添加对应的路由：

```ruby
get 'log_in' => 'sessions#new'
get 'log_out' => 'sessions#destroy'
resources :sessions
```

创建一个登录表单

```ruby
#view/sessions/new.html.erb
<h1>Log in</h1>

<%= form_tag("/sessions", method: 'create') do %>
  <div class="field">
    <%= label_tag :email %>
    <%= text_field_tag :email %>
  </div>
  <div class="field">
    <%= label_tag :password %>
    <%= password_field_tag :password %>
  </div>
  <div class="actions"><%= submit_tag "Log in" %></div>
<% end %>

<figure>
    <img src="/images/20150820-02.png">
</figure>
```

用户在登录的时候，需要去验证邮箱是否注册和密码输入是否正确，`has_secure_password` 提供了校验密码的方法 `authenticate`，同时也需要将用户的 id 存入 session 当中，用于记录用户的状态。退出操作需要将玩家的 id 从 seesion 中清除。

```ruby
class SessionsController < ApplicationController
  def new
  end

  def create
    user = User.find_by(email: params[:email])
    if user and user.authenticate(params[:password])
      session[:user_id] = user.id
      redirect_to user_path(user), :notice => "Logged in!"
    else
      flash.now.alert = "Invalid email or password"
      render 'new'
    end
  end

  def destroy
    session[:user_id] = nil
    redirect_to log_in_path, :notice => "Logged out!"
  end

end
```

如果用户登录成功，则显示用户相关的信息：

<figure>
    <img src="/images/20150820-03.png">
</figure>

如果失败，则给出错误信息：


<figure>
    <img src="/images/20150820-04.png">
</figure>