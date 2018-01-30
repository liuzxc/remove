---
layout: post
title: tmux 快捷键
excerpt: 更改tmux默认快捷键
categories: blog
comments: true
share: true
---


#### 更改tmux默认快捷键

```sh
#cat .tmux.conf
unbind C-b
set -g prefix C-a
bind R source-file ~/.tmux.conf
```

#### 命令

* tmux new -s new_session #创建一个新的会话
* tmux attach -t new_session ＃进入指定会话(new_session)

#### 快捷键

* control+a / &: 退出当前会话
* control+a / d: 退出 tumx，并保存当前会话，这时，tmux 仍在后台运行，可以通过 tmux attach 进入 到指定的会话
* control+a / s: 以菜单方式显示和选择会话
