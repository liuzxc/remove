---
layout: post
title: ruby inject的用法
excerpt:
categories: blog
comments: true
---

inject方法可以帮助你写出更简短的代码，唯一的缺点就是如果看你代码的人不太熟悉inject，那么对方就很难理解你代码的意图。

[inject的用法](http://biyeah.iteye.com/blog/1286449)

在公司的一个项目开发过程中，我遇到了这样一个方法，该方法是用来统计当前玩家
拥有的兵种的数量

{% highlight ruby linenos %}

def current_armies
{
    army1: self.army1,
    army2: self.army2,
    army3: self.army3,
    ...
    army18: self.army18
 }
end

{% endhighlight %}

在每次添加新兵种的时候，我都要更改这个方法，这种做法显得太过死板，而且代码很不好看，我想到了重写这个方法。

{% highlight ruby linenos %}
def current_armies
    army_info = {}
    ALL_ARMIES_TYPE.each { |type| army_info.merge({type.to_sym => self.send(type)}) }
    army_info
end
{% endhighlight %}

现在原来的18行代码只需要5行就可以实现，而且每次添加新兵种的时候我都不需要来更改这个方法，good！

但是army_info这个变量总是让我觉得不舒服，那有没有更好的方法来消除这个变量，让代码变得更简短呢？ inject可以帮助我们实现。

{% highlight ruby linenos %}
def current_armies
    ALL_ARMIES_TYPE.inject({}) { |result, element| result.merge({type.to_sym => self.send(type)}) }
end
{% endhighlight %}

haha! 现在不需要army_info这个变量了，而且一行代码就可以实现目标，是不是很棒呢！