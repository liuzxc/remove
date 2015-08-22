---
layout: post
title:  Rails 应用开发笔记（四）
excerpt: 实现简单的用户访问权限控制功能
comments: true
share: true
categories: articles
---

用户访问权限控制是 web 开发中最基础也是最重要的功能，你不会希望用户可以随意更改其他用户的信息。在
 Rails 的世界中，devise 无疑是开发者的首选，它提供了一整套的解决方案，但是对于小的项目来说，devise
显得过于复杂，在本应用的开发过程中，我们采用了最基础也是最简单的方式来实现该功能。

#### 添加admin字段

毫无疑问，实现该功能的最简单方式就是给 User 增加一个 admin 字段来表示该用户是否是 admin 用户，在 mongoid
中，添加新的字段是非常简单的：

`field :admin, type: Boolean, default: false`

然后通过以下方式授予用户admin权限

`user.update_attribute :admin, true`

#### 在项目中添加权限控制

在没有访问限制之前，任何人都可以查看和编辑用户信息，这样做显然是不安全和不合理的，所以针对用户信息的
编辑和显示这样的操作，需要保证用户当前是登录状态且有足够的权限。因此，需要实现用户是否登录和是否是admin
user 这两个方法：

{% highlight ruby %}
class ApplicationController < ActionController::Base
  ...
  helper_method :admin?

  def require_admin
    if not admin?
      flash[:error] = "您没有权限操作！"
      redirect_to home_path
    end
  end

  def require_login
    if not logged_in?
      flash[:error] = "请登录！"
      redirect_to log_in_path
    end
  end

  private
  ...
  def logged_in?
    current_user.nil? ? false : true
  end

  def admin?
    current_user.admin
  end
end
{% endhighlight %}

require_login 方法作用是验证当前玩家是否是登录状态，如果不是的话，将请求重定向到登录页面。require_admin
方法用来判断玩家是否是管理员账户，并将其设置为 helper_method

>什么是 helper_method？
>如果将方法设置为 helper_method，那就意味着该方法既可以在 controller 中使用，也可以在 view 中使用，这样会
>大大提高编码效率，减少视图层的重复代码，使代码更加清晰。

当用户在对自己的信息进行查看和编辑之前，我们需要验证该用户是否处于登录状态，我们需要在 User controller
中添加以下代码：

{% highlight ruby %}
before_action :require_login, expect: [:new, :create]
{% endhighlight %}

 before_action 的意思是动作在控制层方法执行之前执行，expect 参数的意思是除开数组中的方法。该段代码
 的意思是用户在进行信息的查看（show, index)，更新(update)和删除(destroy)操作之前，需要运行 require_login
 方法验证当前用户是否登录，只有登录的用户才能进行操作，而用户的创建（new,create)需要跳过验证。

> before_action 和 before_filter 的区别
> before_action 和 before_filter其实是一个东西，只是在rails4.0之前使用的是before_filter，之后改成了
before_action，相当于功能一样，只是名字不同而已。

我们希望普通用户只更新自己的信息，而没有权限去更改别人的信息，只有管理员才可以这样做，但是用户组信息页面
却显示可以对所有用户进行更新和删除操作，我们需要采取措施，对普通用户屏蔽掉更新和删除按钮：

{% highlight ruby %}
<% if admin? %>
  <td><%= link_to 'Show', user %></td>
  <td><%= link_to 'Edit', edit_user_path(user) %></td>
  <td><%= link_to 'Destroy', user, method: :delete, data: { confirm: 'Are you sure?' } %></td>
<% end %>
{% endhighlight %}

只有管理员才能看到显示用户详细信息和更新删除按钮。


一个简单的访问权限控制功能就实现了，虽然有一些漏洞，之后会逐渐完善，但是我们从中主要是学习过滤器和 helper_method 的用法。已经深夜两点了，洗洗睡吧！