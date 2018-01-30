---
layout: post
title: Supervisor 使用简介
excerpt: 使用 supervisor 管理进程
categories: blog
comments: true
share: true
---

[Supervisor](http://supervisord.org/)是一个 Python 开发的 client/server 系统，可以管理和监控类 UNIX 操作系统上面的进程。它可以同时启动，关闭多个进程，使用起来特别的方便。


#### 组成部分

supervisor 主要由两部分组成：

1. supervisord(server 部分)：主要负责管理子进程，响应客户端命令以及日志的输出等；
2. supervisorctl(client 部分)：命令行客户端，用户可以通过它与不同的 supervisord 进程联系，获取子进程的状态等。

#### 安装

可以直接使用 pip 安装：

`sudo pip install supervisor`

如果是 Ubuntu 系统，也可以使用 `apt-get`

#### 配置

安装完成之后，可以运行 `echo_supervisord_conf` 生成默认的配置文件：

`echo_supervisord_conf > supervisord.conf`

然后可以通过 supervisord 命令启动 supervisord.

```bash
$ ps aux | grep super
vagrant   1800  0.0  2.4  53680 12140 ?        Ss   06:33   0:00 /usr/bin/python /usr/bin/supervisord
vagrant   1813  0.0  0.1  10432   668 pts/0    S+   06:37   0:00 grep --color=auto super
```

我们可以看到 supervisord 已经被启动了， 然后进入 supervisorctl 的 shell 界面。

```bash
$ supervisorctl
supervisor> status
supervisor>
```

由于目前没有添加任何需要管理的进程，所以 status 没有输出人和结果，接下来我们添加一个需要管理的进程
(以启动一个 celery 的 worker 为例)：

```bash
[program:celeryd]
command=celery worker --app=task -l info ; 启动命令
stdout_logfile=/var/log/supervisor/celeryd_out.log ; stdout 日志输出位置
stderr_logfile=/var/log/supervisor/celeryd_err.log ; stderr 日志输出位置
autostart=true ; 在 supervisord 启动的时候自动启动
autorestart=true ; 程序异常退出后自动重启
startsecs=10 ; 启动 10 秒后没有异常退出，就当作已经正常启动
```

然后运行以下命令更新配置并启动进程：

```bash
$ supervisorctl reread (只更新配置文件)
celeryd: available

$ supervisorctl update (只启动有改动的进程)
celeryd: added process group

$ supervisorctl status
celeryd                          RUNNING    pid 1919, uptime 0:00:18
```

我们看到 celery worker 已经被成功启动了。你可以使用不同的命令来控制进程的启动和关闭。

```bash
$ supervisorctl stop celeryd
celeryd: stopped
$ supervisorctl start celeryd
celeryd: started
$ supervisorctl restart celeryd
celeryd: stopped
celeryd: started
```

把所有的配置文件都放在 `supervisord.conf` 并不是个好主意，一旦管理的进程过多，就很麻烦。所以一般都会
新建一个目录来专门放置进程的配置文件，然后通过 include 的方式来获取这些配置信息

```bash
[include]
files = /etc/supervisor/conf.d/*.conf
```

然后在目录 `/etc/supervisor/conf.d` 下新建一个配置文件 `celery.conf`, 配置信息与上面的一致，效果
是一样的。

#### 命令详解

1. supervisord: 初始启动Supervisord，启动、管理配置中设置的进程;
2. supervisorctl stop(start, restart) xxx，停止（启动，重启）某一个进程(xxx);
3. supervisorctl reread: 只载入最新的配置文件, 并不重启任何进程;
4. supervisorctl reload: 载入最新的配置文件，停止原来的所有进程并按新的配置启动管理所有进程;
5. supervisorctl update: 根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启;
