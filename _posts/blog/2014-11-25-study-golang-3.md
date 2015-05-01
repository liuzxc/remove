---
layout: post
title: go语言学习笔记（三）
excerpt: 理解go语言中的interface
modified:
categories: blog
share: true
---

####Interface

对于长期使用ruby的我来说，go语言的interface是一个让我难以理解的概念，因为ruby中并不存在interface的
说法，那如何让自己更好的理解这个概念呢？我尝试着从这几个角度入手。

* go语言中什么是interface？

`简单的说，interface是一组method的组合，我们通过interface来定义对象的一组行为`

这里有两个需要注意的地方:

1. method的组合.

我们知道go语言只有函数（function）的定义, 而不称作方法

{% highlight go %}
func Sayhi() {
}
{% endhighlight %}

而对于ruby来说，有方法的定义，而不称作函数

{% highlight ruby %}
def method
end
{% endhighlight %}

method和fuction的区别是什么呢？我从stackoverflow上得到的最简单解释是：

`method与对象有关，而fuction与对象无关`

从这点可以看出ruby这种动态语言和go这种静态语言的区别。

那go语言有method么？

答案是肯定的，go语言中的method是这样定义的

{% highlight go %}
func (h Human) Sayhi() {
}
{% endhighlight %}

只要为函数指定一个接收者（receiver），它就可以称作为方法，因为它此时和对象h有关

那ruby中有函数么？

ruby被称作最彻底的面向对象语言，ruby中大部分都是对象，那函数如何存在呢？

ruby元编程一书中说道：

> 尽管ruby中绝大多数东西都是对象，但是块不是。为什么要关心这个呢？设想希望存储一个块供以后执行，这时，
你需要一个对象才可以做到。

stackoverflow中也对此作出了解释：

[ruby-functions-vs-methods](http://stackoverflow.com/questions/928443/ruby-functions-vs-methods)

2. 定义对象的一组行为

go语言作为一种静态语言，对象从何而来呢？

go语言虽然不是OOP，但是它通过struct进行了面向对象的设计。

go语言的struct类型就相当于ruby的class

{% highlight go %}
type Human struct {
	name string
	ane int
}
h := Human{"jason", 25} // h就是Human类型的变量
{% endhighlight %}

* go语言中interface的作用是什么？

go通过interface实现了duck—typing




