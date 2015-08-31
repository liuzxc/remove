---
layout: post
title:  Rails 应用开发笔记（九）
excerpt: 收藏功能的实现
comments: true
share: true
categories: articles
---

如果遇到自己的喜欢的文章，收藏功能必不可少，这次做收藏功能化了我不少时间，一方面是不太熟悉 mongoid document 之间的关联，它与 Active Record 关联有些不同，另外就是不熟悉 Ajax 和 jQuery 的使用，还好花了一个周末的时间总算好是搞定了，不过 Ajax 和 jQuery 的使用我会在下节来详细描述，本篇还是主要介绍收藏功能的实现。

首先是建立关联关系，一个用户会收藏多篇文章，每篇文章又可以被多个用户收藏：

{% highlight ruby %}
class User
  has_many :articles, dependent: :destroy
  has_many :favorites
end

class Article
  belongs_to :user
  has_many   :favorites
end

class Favotite
  belongs_to :user
  belongs_to :article
end
{% endhighlight %}

我们可以对文章进行收藏，也可以取消收藏，因此需要两个控制方法来实现收藏和取消收藏的功能：

{% highlight ruby %}
#app/controller/favorites_controller.rb
class FavoritesController < ApplicationController
  def create
    @article = Article.find(params[:article_id])
    current_user.favorites.create(article_id: params[:article_id])
    redirect_to :back
  end

  def destroy
    favorite = Favorite.find(params[:id])
    @article = favorite.article
    favorite.destroy
    redirect_to :back
  end

end
{% endhighlight %}

> `redirect_to :back` 的意思是返回到前一个页面，所以玩家点击收藏或取消收藏的链接时，不会跳转到新的页面，而是依然回到原页面。

然后添加相应的路由：

{% highlight ruby %}
resources :favorites, only: [:create, :destroy]
{% endhighlight %}

为了让代码清晰，收藏标签的代码使用了一个好单独的 partial:

{% highlight erb %}
＃app/views/favorites/_favorite_link.html.erb
<% if favorite = current_user.favorites.where(article_id: @article.id).first %>
  <%= link_to favorite_path(favorite), method: :delete do %>
    <span class="glyphicon glyphicon-bookmark"></span>
  <% end %>
<% else %>
  <%= link_to favorites_path(article_id: @article.id), method: :post do %>
    <span class="glyphicon glyphicon-bookmark favorite_color"></span>
  <% end %>
<% end %>
{% endhighlight %}

这样一来，一个简单的收藏功能就实现了。但是该功能存在一个致命的缺陷，即每次点击收藏或取消收藏链接的时候，整个页面都会被刷新，这显然是无法接受的，这时就要用到 Ajax 大显身手了，应用中有很多地方都要用到 Ajax 这项技术，因此整理之后我会用单独的篇幅来描述。