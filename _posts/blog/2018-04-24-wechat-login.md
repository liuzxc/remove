---
layout: post
title: 本地测试微信登录的技巧
excerpt: AppID、redirect_uri 参数错误
categories: blog
comments: true
share: true
---

最近调试一个 Rails 项目，需要在本地搭建开发环境，这个项目没有注册功能，只能够通过微信扫码登录，
我在这个地方遇到不少问题，发现网上有很多的朋友也遇到过类似的问题，便想在这个 blog 中总结一下解决的
方案，希望对不熟悉微信登录的朋友有帮助！


### 域名和端口映射

Rails server 启动之后通常跑在 `localhost` 的 3000 端口，这个时候扫码通常会遇到 “AppID 参数错误“:

![AppID 参数错误](https://image.ibb.co/mnkndc/2018_04_24_9_07_18.png)

如果你读过微信开发平台的文档，你应该知道这是由于不合法的 AppID 导致的原因。在开发微信登录功能之前，你应该
在微信开发平台提交应用审核，审核通过后会分配给你一个 AppID 和 AppSecret。但是我并没有注册微信开发平台，但
我可以把线上已经生成的 AppID 和 AppSecret 拿到本地来用，这样也是可以的。果然，更改了 AppID 和 AppSecret 后，
上面的错误消失了，但是却遇到了新的问题——"redirect_uri 参数错误"。

![redirect_uri 参数错误](https://image.ibb.co/cUwqWx/2018_04_24_9_22_39.png)

redirect_uri 是获取临时票据（code) 的接口所用到的参数，redirect_uri 的域名与审核时填写的授权域名一致，显而易见，
本地开发用到的域名是 `localhost`，而审核时填写的域名是网站的正式域名（例如：`www.example.com`)，二者明显不一样。我看
网上有几种解决方案，一是你可以部署一个测试服务器，绑定一个测试域名来模拟环境，这种办法当然比较笨，还有一种简单的方式
就是做域名映射，把 `www.example.com` 映射到 `127.0.0.1` 上来，访问 `www.example.com` 其实是访问本地。

编辑 /etc/hosts 文件

```sh
....
#127.0.0.1  localhost
127.0.0.1   www.example.com
255.255.255.255 broadcasthost
::1             localhost
....
```

尝试打开 `www.example.com:3000`，重新扫码尝试，依然是 redirect_uri 参数错误，这又是怎么回事呢？我们可以打开 Chrome 的调试功能，找到获取 CODE 的
请求:

> https://open.weixin.qq.com/connect/qrconnect?appid=xxxxxxxxx&scope=snsapi_login&redirect_uri=http://www.example.com:3000/auth/open_wechat/callback&state=&login_type=jssdk&self_redirect=default

这里你会注意到，redirect_uri 是带有端口的，而审核填写的域名是不能带端口的，默认80端口，因此我们需要尝试让 rails 启动的时候指定80端口，如果你尝试
`rails server -p 80` 一定会遇到“端口已被占用”的错误，一般情况下，Mac 上 80 端口是被 apache 占用的，所以你需要更改 apache 的默认端口或合适关闭 apache
服务（这里也可以将80端口映射到3000端口）。

> 修改 apache 配置文件 `/private/etc/apache2/httpd.conf` 更改端口，并重启 apache 服务（`sudo apachectl restart`）

然后重新在80端口启动 `rails server`，点击微信登录，哈哈! 错误消失，二维码出现了！拿出手机扫码，看着网页跳转，然后登录成功......

### 禁止 Chrome 自动跳转 https

你有可能会遇到这样的问题：如果你的网站支持 https 访问，当你本地设置域名映射后，网址会自动跳转 https，导致你无法通过 http 访问，这样你在本地就没有办法调试。
解决的方案是：

1. 在 Chrome 地址栏输入： `chrome://net-internals/#hsts`
2. 在 `Delete domain security policies` 填写上域名（例如：`www.example.com`），点击 “delete" 按钮

这样设置以后，网站就不会自动跳转 https 了。

> 参考链接：[网站应用微信登录开发指南](https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&id=open1419316505&token=584f7ae1f2dea1648557f1c27ceef97ea16d17df&lang=zh_CN)
