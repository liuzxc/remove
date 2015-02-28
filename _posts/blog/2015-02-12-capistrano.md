---
layout: post
title: 使用Capistrano 3部署rails应用
excerpt: Capistrano是Rails App最常用的自动化部署工具，本篇文章将介绍Capistrano 3的安装及配置
categories: blog
comments: true
share: true
---

[Capistrano官网](http://capistranorb.com/)

相比于`Capistrano 2`， `Capistrano 3`做了大量的改进，但是官网的文档实在是太过简略，我花了大量的时间去熟悉`Capistrano 3`的使用，整理了一下相关的学习资料，这篇博客会详细介绍`Capistrano 3`的安装及配置。

#####Installation

添加以下代码到`Gemfile`

{% highlight ruby %}
group :development do
	gem 'capistrano-rails'
    #gem 'capistrano' #没有必要在此处再添加这个包，因为capistrano-rails依赖于capistrano，它会自动安装相关的依赖，除非你要指定特定的版本
end
{% endhighlight %}

`capistrano-rails`这个gem包针对`Ruby on Rails`项目做了专门的设计，包括对`Asset Pipeline`和`Database Migration`的支持。

> https://github.com/capistrano/rails/blob/master/README.md

通过`gem dependency`命令, 你可以查看某个gem包依赖于哪些其他的gem包。

{% highlight ruby %}
gem dependency capistrano-rails --reverse-dependencies

Gem capistrano-rails-1.1.2
  capistrano (~> 3.1)
  capistrano-bundler (~> 1.1)
{% endhighlight %}

#####Structure

Capistrano定义了严格的目录结构来管理源码和其他的相关数据

<figure>
    <img src="/images/201502291745.png">
</figure>

* `current`是一个指向最新release的链接，只有在成功的部署之后才会被更新；
* `releases`保存了所有以时间戳命名的部署文件夹；
* `repo`保存了所有版本控制系统的配置；
* `revisions.log`纪录每一次的部署或回滚。
* `shared`包含一些文件和目录，这些文件和目录会被链接到每一个版本，例如想数据库配置和某些敏感信息，包括一些持久化的数据都可以放在这里。

#####Preparing Your Application

######Move secrets out of the repository

像`database.yaml`这样的敏感文件是不应该放在版本控制系统中的，如果你的源码泄漏，很容易导致你的数据库遭受攻击，所以你需要把它排除在版本控制系统之外。

{% highlight bash %}
echo config/database.yml >> .gitignore
{% endhighlight %}

一般情况下，可以用脚本将文件传输到远程服务器的特定目录，在创建数据库的时候再到该目录去读取相应的配置。

######Initialize Capistrano in your application

{% highlight bash %}
$ cd my-project
$ cap install
{% endhighlight %}

<figure>
    <img src="/images/201502291747.png">
</figure>

* `Capfile`会自动的include一些tasks，你可以根据自身项目的需要去选择那些task时需要的。
* `deploy.rb`: 配置全局变量和任务（production和staging均会使用的）
* `production.rb`和`staging.rb`: 定义各自使用的变量和任务
* `lib`目录: 可以在该文件夹下定义任务

未完待续



