---
layout: post
title: Ruby 中的一些基本概念
excerpt: 包括访问权限, mixin, code load 等
categories: blog
comments: true
share: true
---

* [Public VS Protected VS Private](#private)
* [Require VS Load](#require)
* [Include VS Extend](#include)
* [Dup VS Clone](#clone)

#### Public VS Protected VS Private
{: #private}

1. Public 方法可以在任何地方被调用，没有任何限制。方法默认是 public 的，但有两个例外，initialize 方法是 private 的，
在类外定义的“全局”方法也是Object对象的私有方法；
2. Private 方法是实现一个类时使用的内部方法。它只能被这个类或子类的实例方法所调用。它不能被一个显式的接收者所调用，只能被
self所隐式调用。这就意味着私有方法只能在当前对象的的上下文中被调用，你不能调用另外一个对象的私有方法。类的私有方法可以被子类继承，子类可以调用
并覆盖父类的私有方法；
3. Protected 方法与私有方法的相似之处是它也只能在该类或子类的内部被调用，不同之处在于它可以被该类的实例显式调用。

根据实例来分析：

```ruby
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
```

如果在此处把 private 改成 protected，让我们再来看看结果是什么样的:

```ruby
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

```

从上面的例子可以看出，private 和 protected 的区别并不是很大，它们都不能在类定义外被显式的接收者
所调用，但在方法定义中可以被调用，只是在方法定义中，private 也不能被显式调用，但是 protected 可以，
并且 protected 可以被属于同一类或子类的其他对象所调用，但 private 却不可以。

#### Require VS Load
{: #require}


##### Require

require.rb:

```ruby
def test
    puts "hello, world"
end
test
```

test.require.rb:

```ruby
puts require "./require.rb"
puts require './require.rb'
```

output：

```ruby
hello, world
true
false
```

##### Load

load.rb:

```ruby
def test
    puts "hello, world"
end
test
```

test.require.rb:

```ruby
puts load "./load.rb"
puts load "./load.rb"
```

output：

```ruby
hello, world
true
hello, world
true
```

从上面两个例子可以看出，require 和 load 的区别是：如果多次加载同一文件，require 只加载一次，而 load 会加载多次。

#### Include VS Extend
{: #include}


定义一个 module A, class Test1, class Test2:

```ruby
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
```

output:

```ruby
Test1.test => NoMethodError: private method `test' called for Test1:Class
Test1.new.test => hello,world
Test2.test => hello,world
Test2.new.test => NoMethodError: private method `test' called for #<Test2:0x007fa44287bf98>
```

因此，我们可以看出，如果在一个类中 include 一个模块，模块中的方法将成为类的实例方法，如果 extend 一个
模块，模块中的方法将成为类的类方法。

我们再定义一个 module B, class Test3:

```ruby
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
```

如果类中 include 了多个模块，并且模块中有相同的方法，后加载的会覆盖之前加载的方法。

#### Dup VS Clone

这两个方法分配一个调用者所属的实例，然后把调用者的所有实例变量及修改都拷贝到新创建的对象中。
dup 和 clone 用的都是浅拷贝，即如果实例变量包含对其他对象的引用，那么拷贝的是引用而不是对象本身。
它们的区别是：

* clone 比 dup 拷贝更彻底，它还拷贝调用者的单键方法。

```ruby
class A
end

a = A.new

def a.play
   puts "play"
end

a.play => "play"
b = a.clone
b.play => "play"
c = a.dup
c.play => "undefined method `play' for #<A:0x007fcbb184eed0> (NoMethodError)"
```

> 单键方法是指只对单个对象生效的方法，类方法是特殊的单键方法

* clone 保留对象 frozen 状态， 而 dup 不会

```ruby
class Foo
  attr_accessor :bar
end
o = Foo.new
o.freeze # 防止对象被修改

o.dup.bar = 10   # succeeds
o.clone.bar = 10 # RuntimeError: can't modify frozen Foo
```
