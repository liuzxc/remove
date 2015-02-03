---
layout: post
title: go语言学习笔记（三）
modified:
categories: blog
---

常用package解释

#### os

{% highlight go %}
//打开指向标准输入输出和标准错误文件描述符的文件
os.Stdin
os.Stdout
os.Stderr
{% endhighlight %}


#### bufio

{% highlight go %}
reader := bufio.NewReader(os.Stdin) //获取标准输入
input, _ := reader.ReadString('\n') //读取字符，直到遇到换行符结束
fmt.Println(input)
{% endhighlight %}



