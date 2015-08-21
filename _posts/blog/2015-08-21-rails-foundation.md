---
layout: post
title: Rails 开发基础（一）
excerpt:
categories: blog
comments: true
share: true
---

#### Flash

flash 提供了一种可以在不同 actions 之间传递数据的一种方式，所传递的数据是主数据类型（即String, Array, Hash等）。任何放在 flash 中的数据均会暴露给下一个 action 然后被清除掉，非常适合用来显示通知（notices）和警告（alerts）。

{% highlight ruby %}
flash.alert = "You must be logged in"
flash.notice = "Post successfully created"
{% endhighlight %}

##### flash.now

flash.now 和 flash 的不同之处在于 flash.now 只会对当前的 action 生效，不会传递给下一个 action。

{% highlight ruby %}
flash.now.alert = "Beware now!"
# Equivalent to flash.now[:alert] = "Beware now!"
flash.now.notice = "Good luck now!"
# Equivalent to flash.now[:notice] = "Good luck now!"
{% endhighlight %}

> 参考链接：
> http://api.rubyonrails.org/classes/ActionDispatch/Flash/FlashHash.html
> http://api.rubyonrails.org/classes/ActionDispatch/Flash.html
> http://guides.rubyonrails.org/action_controller_overview.html