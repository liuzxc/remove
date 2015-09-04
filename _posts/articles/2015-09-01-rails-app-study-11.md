---
layout: post
title:  Rails 应用开发笔记（十一）
excerpt: 使用 GitHub 帐户登录应用
comments: true
share: true
categories: articles
---

现在很多的应用都支持第三方账户登录，这样可以简化用户登录的流程，为用户带来极大的便利，GitHub 是我
最喜欢也是使用最频繁的网站之一，所以花了一天左右的时间来实现 GitHub 帐户登录应用，我并没有使用 [omniauth](https://github.com/intridea/omniauth) 和 [warden](https://github.com/hassox/warden) 这样的认证框架，一方面是因为我想自己动手实现，另一方面是为了避免代码变的复杂和臃肿。GitHub 官方提供了文档介绍如何接入 GitHub 用户认证，这使开发变的相对容易。

> [Basics of Authentication](https://developer.github.com/guides/basics-of-authentication/)

认证主要分为三步：

1. 注册应用
2. 用户授权
3. 用户登录

#### 注册应用

首先，你需要在 GitHub 上注册你的应用，它会提供一个注册页面，你需要填上应用的相关信息，主要是 `Homepage URL` 和 `Authorization callback URL`，这个可以根据你的喜好来填写，没有严格的限制，
GitHub 会为你的应用生成一个 Client ID 和 Client Secret。

<figure>
    <img src="/images/20150901-01.png">
</figure>

这两个值非常重要，我们需要用它去访问 GitHub API。它们应该被存储在环境变量中，而不是被硬编码或放在版本控制库中，这样做是不安全的，但是在开发环境下，为了方便，我写在了配置文件中（记得在开发环境下不要这样做）：

{% highlight ruby %}
#config/environments/development.rb
ENV['GH_BASIC_CLIENT_ID'] = '00afc453888063ee1c48'
ENV['GH_BASIC_SECRET_ID'] = 'fe2b6a4057deb58ffd0ffa8299bada9951b79ba0'
{% endhighlight %}

#### 用户授权

GitHub 的示例代码采用的是 Sinatra 这个 web 框架，虽然我没怎么用过 Sinatra，但是很容易转化为 Rails 的实现。

{% highlight ruby %}
#app/controller/github_controller.rb
class GithubController < ApplicationController

  CLIENT_ID = ENV['GH_BASIC_CLIENT_ID']
  CLIENT_SECRET = ENV['GH_BASIC_SECRET_ID']

  def index
    render :index, :locals => {:client_id => CLIENT_ID}
  end
end
{% endhighlight %}

render 方法带 locals 参数的意思是渲染一个带有参数的模版，即在 index.html.erb 模版上可以使用 client_id 这个变量。

{% highlight erb %}
#app/views/github/index.html.erb
<html>
  <head>
  </head>
  <body>
    <p>
      Well, hello there!
    </p>
    <p>
      We're going to now talk to the GitHub API. Ready?
      <a href="https://github.com/login/oauth/authorize?scope=user:email&client_id=<%= client_id %>">Click here</a> to begin!</a>
    </p>
    <p>
      If that link doesn't work, remember to provide your own <a href="/v3/oauth/#web-application-flow">Client ID</a>!
    </p>
  </body>
</html>
{% endhighlight %}

创建一个 index 页面用于引导用户访问 GitHub API，授权应用获取用户数据。scope 参数可以定义用户授权给应用的访问权限，你可以设置授予应用不同级别的权限，相关定义可以在 GitHub 的官方文档查到，但一般情况下，应用只需要获取用户的个人信息和邮箱即可。因此 `user:email` 的意思就是应用申请获取用户的个人信息和邮箱地址。

> [scope 定义](https://developer.github.com/v3/oauth/#scopes)

添加路由：

{% highlight ruby %}
#config/routes.rb
get '/github' => 'github#index'
{% endhighlight %}

在浏览器输入链接：`http://localhost:3000`

<figure>
    <img src="/images/20150901-02.png">
</figure>

然后点击 `Click here`，页面会转到github的应用授权页面：

<figure>
    <img src="/images/20150901-03.png">
</figure>

点击 `Authorize application`，授权应用获取你的 GitHub 帐户数据，然后页面会报404错，为什么呢？因为你没有为回调地址设置路由，因此，我们需要添加路由和回调方法：

{% highlight ruby %}
#config/routes.rb
get '/github/callback' => 'github#callback'
{% endhighlight %}

{% highlight ruby %}
#app/controller/github_controller.rb
def callback
  # get temporary GitHub code...
  session_code = request.env['rack.request.query_hash']['code']

  # ... and POST it back to GitHub
  result = RestClient.post('https://github.com/login/oauth/access_token',
                          {:client_id => CLIENT_ID,
                           :client_secret => CLIENT_SECRET,
                           :code => session_code},
                           :accept => :json)

  # extract the token and granted scopes
  access_token = JSON.parse(result)['access_token']
end
{% endhighlight %}

回调方法的作用是获取 access_token，用户授权成功以后，GitHub 会提供一个临时的 code 值，你需要把这个值又 post 回 GitHub 用于交换 access_token。获取 access_token 以后就可以发送请求给 github 获取用户信息了。

> [RestClient](https://github.com/rest-client/rest-client)

#### 用户登录

开发到这里的时候，我遇到一个问题：虽然获取了用户 GitHub 账户的信息，但是这些信息只是一个 Hash，并不是一个 user 的实体，它没有办法登录到我的应用进行各种操作。因此，我需要为 GitHub 帐户创造一个在应用中对应的的用户记录，用户的信息就是 GitHub 账户的信息，并把它们保存在数据库中，这样的话我就可以进行登录，创建文章等各种操作。

拿到 access_token 之后，我们可以把它保存在 session 中。之所以把 access_token 放入 session，原因是 access_token 并不会过期，除非被撤销或删除，因此在认证的时候不用每次都获取新的 access_token。

{% highlight ruby %}
def callback
  ...
  result = RestClient.post('https://github.com/login/oauth/access_token',
                          {:client_id => CLIENT_ID,
                           :client_secret => CLIENT_SECRET,
                           :code => session_code},
                           :accept => :json)
  session[:access_token] = JSON.parse(result)['access_token']
  redirect_to github_path
end
{% endhighlight %}

添加 authenticated? 方法用于判断 access_token 是否已经获取

{% highlight ruby %}
def authenticated?
  session[:access_token]
end
{% endhighlight %}

authenticate! 方法取代 index 方法用于引导用户认证

{% highlight ruby %}
def authenticate!
  render :index, :locals => {:client_id => CLIENT_ID}
end
{% endhighlight %}

而index方法的作用是完成用户登录：

{% highlight ruby %}
def index
    ...
    auth_result = JSON.parse(auth_result)
    user = User.find_by(github_id: auth_result["id"])
    if not user
      password = SecureRandom.hex(8)
      user_attr = {user_name: auth_result["login"],
                   email: auth_result["email"],
                   github_id: auth_result["id"],
                   avatar_url: auth_result["avatar_url"],
                   location: auth_result["location"],
                   password: password,
                   password_confirmation: password,
                  }
      user = User.create!(user_attr)
    end
    session[:user_id] = user.id
    ...
    redirect_to user_path(id: user.id)
  end
end
{% endhighlight %}

以下就是从 GitHub 获取的用户信息：

{% highlight ruby %}
{
  "login"=>"liuzxc",
  "id"=>1954295,
  "avatar_url"=>"https://avatars.githubusercontent.com/u/1954295?v=3",
  "gravatar_id"=>"",
  "url"=>"https://api.github.com/users/liuzxc",
  "html_url"=>"https://github.com/liuzxc",
  "followers_url"=>"https://api.github.com/users/liuzxc/followers",
  "following_url"=>"https://api.github.com/users/liuzxc/following{/other_user}",
  "gists_url"=>"https://api.github.com/users/liuzxc/gists{/gist_id}",
  "starred_url"=>"https://api.github.com/users/liuzxc/starred{/owner}{/repo}",
  "subscriptions_url"=>"https://api.github.com/users/liuzxc/subscriptions",
  "organizations_url"=>"https://api.github.com/users/liuzxc/orgs",
  "repos_url"=>"https://api.github.com/users/liuzxc/repos",
  "events_url"=>"https://api.github.com/users/liuzxc/events{/privacy}",
  "received_events_url"=>"https://api.github.com/users/liuzxc/received_events",
  "type"=>"User",
  "site_admin"=>false,
  "name"=>"Jason Liu",
  "company"=>nil,
  "blog"=>nil,
  "location"=>"Chengdu",
  "email"=>"lxq_102172@163.com",
  "hireable"=>true,
  "bio"=>nil,
  "public_repos"=>21,
  "public_gists"=>4,
  "followers"=>2,
  "following"=>5,
  "created_at"=>"2012-07-11T05:10:00Z",
  "updated_at"=>"2015-04-17T07:43:12Z"
}
{% endhighlight %}

在获取用户信息以后，首先判断该用户在数据库中是否存在（在 User 表中添加一个 `github_id field` 用于标示是否是 GitHub 用户），如果不存在则创建一个。在这里需要注意的是，create 会触发 validation，因此必须为用户生成随机密码，否则会创建失败。创建好之后，将用户的 id 存入 session 当中，表示当前用户已经登录，然后跳转至用户主页。

<figure>
    <img src="/images/20150901-04.png">
</figure>

这样一来，整个认证过程就完成了。