---
layout: post
title:  Rails 应用开发笔记（十二）
excerpt: 使用 turbolinks 实现网页进度条
comments: true
share: true
categories: articles
---

在做 GitHub 账户登录的过程中碰到一个问题：由于认证过程会接连发送几次 http 请求，在网络不好的情况下，点击登录按钮之后发现没有任何反应，过几秒之后才会进入登录状态。对于一般用户来讲，这样的体验是非常不好的，我们希望有个进度条能够显示当前请求的状态，幸运的是，Rails 自带了 gem 包可以帮助实现这个功能，那就是 turbolinks。

turbolinks 的主要作用是为了加速页面渲染，进度条功能只是它提供的其中的一个功能之一，但是本篇主要介绍如何通过它来实现网页进度条，加速渲染功能可能会在之后介绍道。

> [turbolinks](https://github.com/rails/turbolinks)

如果 turbolinks 3 正式发布之后，也许我这篇文章就没有什么作用了，因为版本3已经默认进度条功能是开启状态，所以完全不需要做任何特殊的配置，然是如果用的是 turbolinks 3 之前的版本，你需要做一些配置，只需要一行代码：

```js
#app/assets/javascripts/application.js
Turbolinks.enableProgressBar();
```

该行代码的意思是打开 Turbolinks 的进度条功能，当重新点击网站页面的时候，进度条就会出现，超 easy！

<figure>
    <img src="http://zippy.gfycat.com/IncomparableDeadlyArctichare.gif">
</figure>

如果想定制进度条的外观，可以通过 CSS 来实现：

```js
#app/assets/stylesheets/application.scss
html.turbolinks-progress-bar::before {
  background-color: red !important;
  height: 5px !important;
}
```

然后点击网站页面，进度条变成红色，并且宽度增加：

<figure>
    <img src="http://zippy.gfycat.com/AdeptGranularArmyant.gif">
</figure>