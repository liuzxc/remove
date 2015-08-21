---
layout: post
title: Rails 开发基础（一）
excerpt:
categories: blog
comments: true
share: true
---

#### Flash

flash提供了一种可以在不同actions之间传递数据的一种方式，所传递的数据是主数据类型（即String, Array, Hash等）。任何放在flash中的数据均会暴露给下一个action然后被清除掉，非常适合用来显示通知（notices）和警告（alerts）。

flash.alert = "You must be logged in"
flash.notice = "Post successfully created"

##### flash.now

flash.now和flash的不同之处在于flash.now只会对当前的action生效，不会传递给下一个action。

flash.now.alert = "Beware now!"
# Equivalent to flash.now[:alert] = "Beware now!"
flash.now.notice = "Good luck now!"
# Equivalent to flash.now[:notice] = "Good luck now!"

> 参考链接：
> http://api.rubyonrails.org/classes/ActionDispatch/Flash/FlashHash.html
> http://api.rubyonrails.org/classes/ActionDispatch/Flash.html
> http://guides.rubyonrails.org/action_controller_overview.html