---
layout: post
title:  Rails 应用开发笔记（十四）
excerpt: 实现软删除（soft delete）功能
comments: true
share: true
categories: articles
---

软删除即不真正删除数据库中的数据，数据是可以恢复的，只是对用户来讲是不可见的。软删除在实际应用中很广泛，除了一般的论坛应用，在微博和微信之类的应用中也随处可见。为什么要有软删除而不是直接删除呢？个人认为主要有两个方面的原因：一是数据本身就是有价值或有潜在价值的东西，二是和数据本身关联的其他数据有很多，比如文章的评论，如果直接删除了文章，那么评论也会随之被删除，某些用户可能并不希望这样。因此，软删除可以很好的解决这个问题。

我想在应用中实现软删除也主要是因为文章和评论不宜直接从数据库中删除，如果一篇文章被作者删除，应用需要告诉收藏这篇文章的用户和评论这篇文章的用户：这篇文章已经被作者删除了；当3楼的评论被作者删除，应用也需要告诉@3楼的其他用户：3楼的评论已经被作者删除。这可以提高用户的体验。

一开始，我对如何实现软删除是没有什么好的思路的，我也知道会有很多的 gem 包可以实现这个功能，但是我还是想手动实现软删除的功能。通过求助 stackoverflow，我得到了一些思路：

1) 为数据库添加一个新字段来表示记录是否被软删除
2) 使用 `ActiveSupport::Concern`

第一个思路很好理解，即在数据库表中添加一个新字段做标志位，如果需要软删除，则设置为1，否则设置为0，也可以填充其他内容。

{% highlight ruby %}
#app/models/comment.rb
class Comment
  ...
  field :deleted_at, type: DataTime
  default_scope ->{ where(deleted_at: nil) }
  ...

  def soft_destroy
    self.update_attributes(deleted_at: Time.now)
  end
  ...
end
{% endhighlight %}

此处设置 deleted_at 为标志位，如果是软删除则设置 deleted_at 的时间为当前时间。设置默认的 scope 是为了在每次查询的时候过滤掉软删除的记录。soft_destroy 是用来取代 destroy 方法。这样一来就基本上实现了软删除的功能，看起来还是比较简单的，对吧😄

但是此处存在一个问题：除了 Comment model 需要添加软删除的功能，Article model 也需要，如果简单的拷贝代码到 。Article model 当中，那就违反了 Rails DRY 原则。我们需要把软删除的功能封装成一个 module，可以让不同的 model 有选择性的调用，此处就要用到 `ActiveSupport::Concern`。

> ActiveSupport::Concern

在 models 的 concerns 目录新添加一个文件 soft_delete.rb，将软删除的功能封装到其中的 SoftDelete module 当中。

{% highlight ruby %}
#app/models/concerns/soft_delete.rb
module SoftDelete
  extend ActiveSupport::Concern

  included do
    field :deleted_at, type: DateTime
    default_scope ->{ where(deleted_at: nil) }

    def soft_destroy
      self.update_attributes(deleted_at: Time.now)
    end
  end
end
{% endhighlight %}

在 Comment model中引用 SoftDelete module：

{% highlight ruby %}
#app/models/comment.rb
class Comment
  include Mongoid::Document
  include SoftDelete
  ...
end
{% endhighlight %}

将 `comments_controller.rb` 中的 destroy 方法替换为 soft_destroy 方法：

{% highlight ruby %}
#app/controllers/comments_controller.rb
def destroy
  @comment.soft_destroy
  ...
end
{% endhighlight %}

修改对应的视图文件，根据 deleted_at 字段判断评论是否被删除：

{% highlight erb %}
#app/views/comments/_show_html.erb
<% @article.comments.unscoped.each_with_index do |comment, index| %>
  <ul class="list-group">
    <li class="list-group-item">
    <% if comment.deleted_at %>
      <h6 class="text-center">抱歉，此条评论已被作者删除</h6>
    <% else %>
      ...
      <div class="media-body">
        <h6 class="media-heading"><%= link_to comment.name, user_path(User.find_by(user_name: comment.name)) %> / <%= "#{comment.floor}楼" %>
          <% if User.find_by(user_name: comment.name) == current_user %>
            <%= link_to '编辑', edit_article_comment_path(comment.article, comment) %>
            <%= link_to '删除', [comment.article, comment], method: :delete %>
          <% end %>
        </h6>
        <%= markdown(comment.content) %>
      </div>
      ...
</div>
{% endhighlight %}

然后来看看效果：

<figure>
  <img src="http://zippy.gfycat.com/HairyMatureAsianpiedstarling.gif">
</figure>


