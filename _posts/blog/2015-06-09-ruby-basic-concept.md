---
layout: post
title: Ruby 中的一些基本概念
excerpt: 包括访问权限, mixin, code load 等
categories: blog
comments: true
share: true
---

#### Public VS Protected VS Private

1. Public 方法可以被任何人调用——无强制的访问限制。一个实例方法默认是 public 的，任何人都可以调用，
但 initialize 方法是 private 的；
2. Protected 方法只可以被类及其子类调用；
3. Private 方法不能被一个显式的接收者所调用，接收者只能是自身(self)。这就意味着私有方法只能在当前
对象的的上下文中被调用，你不能调用另外一个对象的私有方法。类的私有方法可以被子类继承，子类可以调用
并覆盖父类的私有方法；

根据实例来分析：

{% highlight ruby %}
class A
  def test1
    test4
  end

  def test2
    self.test4
  end

  def test3
    A.new.test4
  end

  private
    def test4
      puts "hello, world!"
    end

    def self.test5
      puts "hello, Jason!"
    end
end

a = A.new
a.test1 => hello, world!
#私有方法可以在当前对象的上下文中被调用

a.test2 => NoMethodError: private method `test4' called for #<A:0x007f80b9144c40>
#私有方法不能被显式的接收者调用，即使接收者是self

a.test3 => NoMethodError: private method `test4' called for #<A:0x007f80b914c508>
#私有方法不能被显式的接收者调用,更不能调用其他对象的私有方法

a.test4 => NoMethodError: private method `test4' called for #<A:0x007f80b9144c40>
#同上，私有方法不能被显式的接收者调用

A.test5 => hello, Jason!
#private只作用于实例方法，对类方法是不生效的，如果要让类方法称为私有的，应当使用 private_class_method

{% endhighlight %}

如果在此处把 private 改成 protected，让我们再来看看结果是什么样的:

{% highlight ruby %}
class A
  def test1
    test4
  end

  def test2
    self.test4
  end

  def test3
    A.new.test4
  end

  protected
    def test4
      puts "hello, world!"
    end

    def self.test5
      puts "hello, Jason!"
    end
end

a = A.new
a.test1 => hello, world!
#protected方法可以在当前对象的上下文中被调用

a.test2 => NoMethodError: private method `test4' called for #<A:0x007f80b9144c40>
#protected方法在方法定义中可以被显式的接收者self调用

a.test3 => NoMethodError: private method `test4' called for #<A:0x007f80b914c508>
#protected方法可以被属于同一类或子类的其他对象所调用

a.test4 => NoMethodError: private method `test4' called for #<A:0x007f80b9144c40>
#protected方法在类定义外不能被显式的接收者调用

A.test5 => hello, Jason!
#protected只作用于实例方法，对类方法是不生效的

{% endhighlight %}

从上面的例子可以看出，private 和 protected 的区别并不是很大，它们都不能在类定义外被显式的接收者
所调用，但在方法定义中可以被调用，只是在方法定义中，private 也不能被显式调用，但是 protected 可以，
并且 protected 可以被属于同一类或子类的其他对象所调用，但 private 却不可以。

#### require VS load

##### require

require.rb:

{% highlight ruby %}
def test
    puts "hello, world"
end
test
{% endhighlight %}

test.require.rb:

{% highlight ruby %}
puts require "./require.rb"
puts require './require.rb'
{% endhighlight %}

output：

{% highlight ruby %}
hello, world
true
false
{% endhighlight %}

##### load

load.rb:

{% highlight ruby %}
def test
    puts "hello, world"
end
test
{% endhighlight %}

test.require.rb:

{% highlight ruby %}
puts load "./load.rb"
puts load "./load.rb"
{% endhighlight %}

output：

{% highlight ruby %}
hello, world
true
hello, world
true
{% endhighlight %}

从上面两个例子可以看出，require 和 load 的区别是：如果多次加载同一文件，require 只加载一次，而 load 会加载多次。

#### include VS extend

定义一个 module A, class Test1, class Test2:

{% highlight ruby %}
module A
  def test
     puts "hello,world!"
  end
end

class Test1
  include A
end

class Test2
  extend A
end
{% endhighlight %}

output:

{% highlight ruby %}
Test1.test => NoMethodError: private method `test' called for Test1:Class
Test1.new.test => hello,world
Test2.test => hello,world
Test2.new.test => NoMethodError: private method `test' called for #<Test2:0x007fa44287bf98>
{% endhighlight %}

因此，我们可以看出，如果在一个类中 include 一个模块，模块中的方法将成为类的实例方法，如果 extend 一个
模块，模块中的方法将成为类的类方法。

我们再定义一个 module B, class Test3:

{% highlight ruby %}
module B
  def test
    puts "hello Jason"
  end
end

class Test3
  include A
  include B
end

Test3.new.test => hello Jason

{% endhighlight %}

如果类中 include 了多个模块，并且模块中有相同的方法，后加载的会覆盖之前加载的方法。