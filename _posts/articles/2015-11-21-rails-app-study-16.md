---
layout: post
title:  Rails 应用开发笔记（十六）
excerpt: 使用 Vagrant 构建虚拟开发环境
comments: true
share: true
categories: articles
---

最近在新工作中接触到了 vagrant 这样一个非常好的工具，便想将其应用到自己的应用当中。[vagrant](https://www.vagrantup.com/) 是一款用来构建虚拟开发环境的工具，非常适合 ruby/python 这类语言开发 web 应用，我们可以通过 vagrant 封装一个 Linux 开发环境，无论你使用什么样的操作系统，你都可以在虚拟机里面跑你的代码，与你本地的环境相隔离。在项目里面，这样的工具太有用了，你不会再听到 “代码在我的机器上跑不起来” 这样的抱怨了，是不是很赞呢！接下来我就尝试让自己的应用通过 vagrant 在虚拟机上跑起来。

#### Installation

1. 安装 [vagrant](http://www.vagrantup.com/downloads)
2. 安装 [virtualbox](https://www.virtualbox.org/)

#### UP and RUNNING

vagrant 安装好以后，在项目目录下执行:

```sh
$ vagrant init
```

该命令会生成一个名叫 Vagrantfile 的配置文件：

```sh
...
# Every Vagrant development environment requires a box. You can search for
# boxes at https://atlas.hashicorp.com/search.
config.vm.box = "ubuntu/trusty64"
...
```

这一部分是配置文件的关键，你可以通过它设置你想在虚拟机上跑的操作系统，我使用的是 64位 的 ubuntu， 你也可以根据它提供的链接选择自己喜欢的，然后跑下面这个命令：

```sh
$ vagrant up
```

vagrant 会自动为你安装虚拟环境，安装完成之后你就可以 ssh 上虚拟机了

```sh
$ vagrant ssh
```

现在你就有了自己的虚拟环境了，你可以在上面安装项目所需要的依赖，下面以我自己的应用为例：

#### 配置开发环境

##### 安装 git

```sh
sudo apt-get install git-core
```

##### 安装 rbenv 和 ruby-build

```sh
git clone git://github.com/sstephenson/rbenv.git .
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
source .bash_profile
git clone https://github.com/sstephenson/ruby-build.git
cd ruby-build/
sudo ./install.sh
```

##### 安装 ruby 和 rails

```sh
sudo apt-get install -y libssl-dev libreadline-dev zlib1g-dev build-essential g++ nodejs

rbenv install 2.2.2
rbenv global 2.2.2

gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org/
gem install rails -v 4.2.3
```

##### 安装 mongodb

```sh
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
```

##### start rails server

```sh
bundle install
rails server
```

经过这样一番折腾之后，我的应用就在 vagrant 构建的虚拟机上跑起来了，但是有个问题，怎么在本机上访问跑在在虚拟机上的应用呢？

首先修改 Vagrantfile 的配置：

```sh
...
# Create a forwarded port mapping which allows access to a specific port
# within the machine from a port on the host machine. In the example below,
# accessing "localhost:8080" will access port 80 on the guest machine.
config.vm.network "forwarded_port", guest: 3000, host: 3000
...
```

修改之后需要退出虚拟机，运行 `vagrant reload` 去加载新的配置，不然修改是不会生效的哦。

把 `rails server` 默认绑定的 IP 从 `127.0.0.1` 改为 `0.0.0.0`：

```ruby
#config/boot.rb
ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../../Gemfile', __FILE__)

require 'bundler/setup' # Set up gems listed in the Gemfile.

require 'rails/commands/server'
module Rails
  class Server
    def default_options
      super.merge(Host:  '0.0.0.0', Port: 3000)
    end
  end
end
```

这样一来，这个开发环境就搭建好了。

##### provision

vagrant 提供了一个 provision 的功能，可以在 `vagrant up` 的时候自动安装项目运行所依赖的环境，我们只需要做简单的配置和提供一个脚本：

```sh
# vagrantfile
ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../../Gemfile', __FILE__)
  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
  config.vm.provision "shell", path: 'bootstrap.sh'
```

打开 provision，并制定 shell 脚本 `bootstrap.sh` 用于 provisioning。

```sh
# bootstrap.sh

#!/usr/bin/env bash
sudo update-locale LC_ALL="en_US.utf8"

echo "Install mongodb:"
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/mongodb.list

echo "Add Ruby sources:"
sudo apt-add-repository -y ppa:brightbox/ruby-ng

sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y git ruby2.2 ruby2.2-dev mongodb-10gen nodejs zlib1g-dev build-essential g++ libsqlite3-dev

gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/

sudo gem install bundler
bundle config mirror.https://rubygems.org https://ruby.taobao.org
```

这样一来，在运行 `vagrant up` 的时候会自动为你安装项目所依赖的环境。