---
layout: post
title:  Rails 应用开发笔记（十三）
excerpt: 使用 StackOverflow 帐户登录应用
comments: true
share: true
categories: articles
---

使用 StackOverflow 帐户登录应用和之前 GitHub 帐户登录应用的原理几乎是一模一样的，代码几乎可以重用，本来无需单独做介绍，但是还是遇到一些问题，花了些时间去排查，还是觉得有必要写下来。个人一开始认为 StackOverflow 应该是属于技术比较牛逼的网站，文档应该比较详尽，至少会有几个语言的示例，但是结果相反，API文档写的比较简略，也没有什么示例，某些内容还没怎么看懂，还好原理和接入 GitHub 是一样的，省了很多时间。

Authentication 这一块的文档比较简略，但是条例很清楚，按着它的流程走很容易实现，但是需要注意的是：

1. StackOverflow 的用户认证都是基于 OAuth 2.0 实现的，它提供了两个认证流程，一个是针对服务端应用的`The explicit OAuth 2.0`，一个是针对客户端应用的`The implicit OAuth 2.0`，二者流程不一样，很显然 Rails 应用要使用针对服务端应用的流程。

2. StackOverflow 的邮箱信息是私有的，并不对外开放，因此无法通过它的API获取用户的邮箱

3. 需要注册应用，有几个地方需要注意：如果是在本地测试，OAuth Domain 这一栏需要填域名，而不能是 IP 或者 localhost，可以更改 host 文件指定你想要的域名，比如 `rails.example.com`；Application Website 这栏需要填写完整的 url，包括协议和端口，比如`http://rails.example.com:3000`；如果是服务端应用，不要勾选`Enable Client Side OAuth Flow`

<figure>
    <img src="/images/20150903-01.png">
</figure>

4. 应用注册记录的地址是`http://stackapps.com/apps/oauth/view/＃{client_id}`(注册页面关闭后，死活没找到注册记录在哪里，只有翻网页历史纪录，不得不说 StackOverflow 这个设计真坑爹！)

注意到这几个方面以后，接入 StackOverflow 帐户登录应用的流程就很简单了，可以参照文档，这里就不细说了。

> [Authentication API](https://api.stackexchange.com/docs/authentication)
> [OAuth 2.0 介绍](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)

<figure>
    <img src="http://zippy.gfycat.com/BlindCanineAfricanjacana.gif">
</figure>