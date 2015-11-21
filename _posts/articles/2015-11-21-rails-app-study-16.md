---
layout: post
title:  Rails 应用开发笔记（十六）
excerpt: 使用 Vagrant 构建虚拟开发环境
comments: true
share: true
categories: articles
---

最近在新工作中接触到了 vagrant 这样一个非常好的工具，便想将其应用到自己的应用当中。[vagrant](https://www.vagrantup.com/) 是一款用来构建虚拟开发环境的工具，非常适合 ruby／python 这类语言开发web应用，我们可以通过 vagrant 封装一个 Linux 开发环境，无论你使用什么样的操作系统，你都可以在虚拟机里面跑你的代码，与你本地的环境相隔离。在项目里面，这样的工具太有用了，你不会再听到 “代码在我的机器上跑不起来” 这样的抱怨了，是不是很赞呢！接下来我就尝试让自己的应用通过 vagrant 在虚拟机上跑起来。

#### Installation

1. 安装 [vagrant](http://www.vagrantup.com/downloads)
2. 安装 [virtualbox](https://www.virtualbox.org/)

#### UP and RUNNING

vagrant 安装好以后，在项目目录下执行 vagrant init

{% highlight shell %}
$ vagrant init
{% endhighlight %}

该命令会生成一个名叫 Vagrantfile 的配置文件：

{% highlight shell %}
...
# Every Vagrant development environment requires a box. You can search for
# boxes at https://atlas.hashicorp.com/search.
config.vm.box = "ubuntu/trusty64"
...
{% endhighlight %}

这一部分是配置文件的关键，你可以通过它设置你想在虚拟机上跑的操作系统，我使用的是 64位 的 ubuntu， 你也可以根据它提供的链接选择自己喜欢的。然后跑下面这个命令：

{% highlight shell %}
$ vagrant up
{% endhighlight %}

vagrant 会自动为你安装虚拟环境。安装完成之后你就可以 ssh 上虚拟机了

{% highlight shell %}
$ vagrant ssh
{% endhighlight %}

现在你就有了自己的虚拟环境了，你可以在上面安装项目所需要的依赖，下面以我自己的应用为例：

#### 配置开发环境

##### 安装 git

{% highlight shell %}
sudo apt-get install git-core
{% endhighlight %}

##### 安装 rbenv 和 ruby-build

{% highlight shell %}
git clone git://github.com/sstephenson/rbenv.git .
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
source .bash_profile
git clone https://github.com/sstephenson/ruby-build.git
cd ruby-build/
sudo ./install.sh
{% endhighlight %}

##### 安装 ruby 和 rails

{% highlight shell %}
sudo apt-get install -y libssl-dev libreadline-dev zlib1g-dev build-essential g++ nodejs

rbenv install 2.2.2
rbenv global 2.2.2

gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org/
gem install rails -v 4.2.3
{% endhighlight %}

##### 安装 mongodb

{% highlight shell %}
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
{% endhighlight %}

##### start rails app

{% highlight shell %}
bundle install
rails server
{% endhighlight %}

经过这样一番折腾之后，我的应用就在 vagrant 构建的虚拟机上跑起来了，但是有个问题，怎么在本机上访问跑在在虚拟机上的应用呢？

只要修改 Vagrantfile 的配置就好啦

{% highlight shell %}
...
# Create a forwarded port mapping which allows access to a specific port
# within the machine from a port on the host machine. In the example below,
# accessing "localhost:8080" will access port 80 on the guest machine.
config.vm.network "forwarded_port", guest: 3000, host: 3000
...
{% endhighlight %}

修改之后需要退出虚拟机，运行 vagrant reload 去加载新的配置，不然修改是不会生效的哦。




