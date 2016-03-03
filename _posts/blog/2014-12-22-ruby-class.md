---
layout: post
title: Ruby 中的类和对象
excerpt: ruby
categories: blog
comments: true
share: true
---

最近在ruby－china中闲逛，无意中发现了一位网友提出的问题，这个问题相当有意思，并且促使我写下了这篇博客。

原问题是这样描述的：

{% highlight ruby %}
class Object
  def do_it
    puts "do it"
  end
end

Object.do_it #这个class method 怎么来的？
Object.new.do_it
{% endhighlight %}

简单的几行代码，我竟一时答不上来，不禁汗颜！接触ruby也有一年多了，自认为很熟悉这门语言，但这个问题却把自己打回原型。不过我相信，至少有90%的ruby程序员会被这个问题难住，所以我觉得很有必要研究一下这个问题，让自己，也让别人搞明白，为什么会这样。

首先，我想每个人心中都会有这样一个疑问：

`do_it`很明显是`Object`对象的实例方法，而`Object`是一个类，类怎么可以直接调用其实例方法呢？

我们暂时放下这个问题，去分析一下`Object`这个类，它到底有什么特别之处。

让我们先写一个普通类，看它是否和`Object`类有相似的特性。

{% highlight ruby %}
irb(main):001:0> class Test
irb(main):002:1> def do_it
irb(main):003:2> puts "do it"
irb(main):004:2> end
irb(main):005:1> end
{% endhighlight %}

我们写了一个`Test`类，它也有一个`do_it`方法，我们可以让`Test`类来调用`do_it`，看会发生什么：

{% highlight ruby %}
irb(main):006:0> Test.do_it
NoMethodError: undefined method `do_it' for Test:Class
{% endhighlight %}

正如我们预料的一样，`Test`并不能调用`do_it`这个实例方法，那`Object`为什么可以办到呢?

{% highlight ruby %}
irb(main):007:0> Test.ancestors
=> [Test, Object, Kernel, BasicObject]
irb(main):010:0> Test.superclass
=> Object
{% endhighlight %}

`ancestors`方法可以查看当前类的祖先链，我们可以清楚的看到：`Object`类是`Test`类的超类(正确来说，所有类最终都继承于`Object`)。

<figure>
  <img src="/images/Ruby_object_model.png" alt="image">
  <figcaption>ruby对象模型是理解ruby语言的关键</figcaption>
</figure>

ruby中，可以使用`class`方法查看对象所属的类。

{% highlight ruby %}
irb(main):011:0> a = Test.new
=> #<Test:0x007ffee21212c0>
irb(main):012:0> a.class
=> Test
irb(main):013:0> Test.class
=> Class
{% endhighlight %}

从这里可以看出ruby中一个非常重要的概念：**类自身也是对象**。

既然类自身也是对象，那么我们也可以把`Object`类当作对象。

{% highlight ruby %}
irb(main):014:0> Object.class
=> Class
irb(main):013:0> Test.class
=> Class
{% endhighlight %}

What happened? 和ruby对象模型中描述的一样，`Object`类和`Test`类都属于`Class`类。然后我们再回到之前的代码：

{% highlight ruby %}
Object.do_it
{% endhighlight %}

在这里，`Object`是类还是对象呢？

{% highlight ruby %}
irb(main):024:0> Object.instance_methods
=> [:do_it, :nil?, :===, :=~, :!~, :eql?, :hash, :<=>, :class, :singleton_class, :clone, :dup, :taint, :tainted?, :untaint, :untrust, :untrusted?, :trust, :freeze, :frozen?, :to_s, :inspect, :methods, :singleton_methods, :protected_methods, :private_methods, :public_methods, :instance_variables, :instance_variable_get, :instance_variable_set, :instance_variable_defined?, :remove_instance_variable, :instance_of?, :kind_of?, :is_a?, :tap, :send, :public_send, :respond_to?, :extend, :display, :method, :public_method, :singleton_method, :define_singleton_method, :object_id, :to_enum, :enum_for, :==, :equal?, :!, :!=, :instance_eval, :instance_exec, :__send__, :__id__]
{% endhighlight %}

`instance_methods`方法可以打印出当前类所拥有的所有实例方法，很明显，`do_it`是`Obje1ct`类的实例方法。

对于`Object.do_it`，如果把`Object`当作类来看，很明显类无法直接调用实例方法，但是如果把`Object`当作对象来看，情况就不一样了。

根据ruby的方法查找方式：**沿着祖先链查找**。

我们试着找出`Object`是如何找到`do_it`的方法的。

{% highlight ruby %}
irb(main):026:0> Object.class
=> Class //Object所属的类是Class，Class中没有定义do_it,然后查找超类
irb(main):027:0> Class.superclass
=> Module //Module中也没有定义do_it,继续查找超类
irb(main):028:0> Module.superclass
=> Object //My god! Module的超类竟然是Object，而Object中确实定义了do_it方法
{% endhighlight %}

我们终于找到了do_it方法，一切都真相大白！！！而这一切都如ruby对象模型中描述的一模一样。

简单的一个问题，却折射出学习ruby三个最重要的知识点：

* ruby中，一切皆对象
* 理解ruby对象模型
* 了解ruby查找方法的方式
