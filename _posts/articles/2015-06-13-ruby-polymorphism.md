---
layout: post
title: 多态与 Ruby
excerpt: 介绍 Ruby 多态的实现方式
comments: true
share: true
categories: articles
---

原文链接：https://robots.thoughtbot.com/back-to-basics-polymorphism-and-ruby

多态是面向对象编程最基本的特征之一，但是它到底是什么意思？就其核心而言，对于 Ruby，可以发送相同的消息
给不同的对象，然后得到不同的结果。让我们来看看 Ruby 实现多态的几种方式。

### 继承

实现多态的其中一种方式就是继承。我们用**模版方法**创建一个简单的文件解析器。

首先我们创建一个带有 parse 方法的 GenericParser 类，这个模版类的唯一的方法做的事情的就是抛出一个异常：

{% highlight ruby %}
class GenericParser
  def parse
    raise NotImplementedError, 'You must implement the parse method'
  end
end
{% endhighlight %}

然后我们声明一个继承 GenericParser 类的 JsonParser 类：

{% highlight ruby %}
class XmlParser < GenericParser
  def parse
    puts 'An instance of the XmlParser class received the parse message'
  end
end
{% endhighlight %}

再创建一个继承 GenericParser 类的 XmlParser 类：

{% highlight ruby %}
class XmlParser < GenericParser
  def parse
    puts 'An instance of the XmlParser class received the parse message'
  end
end
{% endhighlight %}

然后运行一个脚本，看看发生了什么：

{% highlight ruby %}
puts 'Using the XmlParser'
parser = XmlParser.new
parser.parse

puts 'Using the JsonParser'
parser = JsonParser.new
parser.parse
{% endhighlight %}

输出结果：

{% highlight plaintext %}
Using the XmlParser
An instance of the XmlParser class received the parse message

Using the JsonParser
An instance of the JsonParser class received the parse message
{% endhighlight %}

我们注意到代码行为的不同依赖于子类接收的 parse 方法。 XML 和 JSON 解析器修改了 GenericParser 的行为。

### 鸭子类型（Duck Typing)

对于静态类型语言，运行时多态（runtime polymorphism）更难以实现。幸运的是，Ruby 可以使用鸭子类型。

我们再次以 XML 和 JSON 解析器为例，去掉继承：

{% highlight ruby %}
class XmlParser
  def parse
    puts 'An instance of the XmlParser class received the parse message'
  end
end

class JsonParser
  def parse
    puts 'An instance of the JsonParser class received the parse message'
  end
end
{% endhighlight %}

现在我们创建一个通用解析器，它可以发送 parse 消息，并且接收一个参数。

{% highlight ruby %}
class GenericParser
  def parse(parser)
    parser.parse
  end
end
{% endhighlight %}

现在我们有一个非常棒的鸭子类型的例子。注意，parse 方法接收一个叫 parser 的变量。此工作所需的唯一事情就是让解析器对象响应解析消息,
幸运的是我们两解析器都可以那样做!

{% highlight ruby %}
parser = GenericParser.new
puts 'Using the XmlParser'
parser.parse(XmlParser.new)

puts 'Using the JsonParser'
parser.parse(JsonParser.new)
{% endhighlight %}

输出结果：

{% highlight plaintext %}
Using the XmlParser
An instance of the XmlParser class received the parse message

Using the JsonParser
An instance of the JsonParser class received the parse message
{% endhighlight %}

我们注意到方法行为的不同依赖于对象接收的消息。这就是多态！

### 装饰模式

我们也可以通过设计模式的使用来实现多态。让我们看一个装饰模式的例子：

{% highlight ruby %}
class Parser
  def parse
    puts 'The Parser class received the parse method'
  end
end
{% endhighlight %}

我们需要修改 XmlParser 类，让它包含一个带有解析器参数的构造方法。

{% highlight ruby %}
class XmlParser
  def initialize(parser)
    @parser = parser
  end

  def parse
    @parser.parse
    puts 'An instance of the XmlParser class received the parse message'
  end
end
{% endhighlight %}

对 JsonParser 做相同的修改：

{% highlight ruby %}
class JsonParser
  def initialize(parser)
    @parser = parser
  end

  def parse
    puts 'An instance of the JsonParser class received the parse message'
    @parser.parse
  end
end
{% endhighlight %}

我们将使用装饰者去创建普通的 XML 和 JSON 解析器，但是在最后一个例子中，我们做的会有一点不同：
都使用装饰者实现运行时多态：

{% highlight ruby %}
puts 'Using the XmlParser'
parser = Parser.new
XmlParser.new(parser).parse

puts 'Using the JsonParser'
JsonParser.new(parser).parse

puts 'Using both Parsers!'
JsonParser.new(XmlParser.new(parser)).parse
{% endhighlight %}

输出结果：

{% highlight plaintext %}
Using the XmlParser
The Parser class received the parse method
An instance of the XmlParser class received the parse message

Using the JsonParser
An instance of the JsonParser class received the parse message
The Parser class received the parse method

Using both Parsers!
An instance of the JsonParser class received the parse message
The Parser class received the parse method
An instance of the XmlParser class received the parse message
{% endhighlight %}

### 多态的优点

多态的其中一个优点是可以简化代码。看看下面的类：

{% highlight ruby %}
class Parser
  def parse(type)
    puts 'The Parser class received the parse method'

    if type == :xml
      puts 'An instance of the XmlParser class received the parse message'
    elsif type == :json
      puts 'An instance of the JsonParser class received the parse message'
    end
  end
end
{% endhighlight %}

由于鸭子类型的优势，我们可以通过移除条件分支逻辑来简化代码。现在我们把特定的解析逻辑放在它们各自的
类当中：

{% highlight ruby %}
class Parser
  def parse(parser)
    puts 'The Parser class received the parse method'
    parser.parse
  end
end

class XmlParser
  def parse
    puts 'An instance of the XmlParser class received the parse message'
  end
end

class JsonParser
  def parse
    puts 'An instance of the JsonParser class received the parse message'
  end
end
{% endhighlight %}

这个例子表明：我们可以利用多态简化代码，同时也符合单一责任原则（Single Responsibility Principle）

在最初的版本中，Parser 类决定使用哪一个解析器，通过实例化去初始化解析器，然后发送解析消息给对象。之后的例子中，
在最初的版本中，Parser 类只需要调用 parse 方法触发解析过程。

多态是面向对象编程的基础之一，但是也容易让人感到疑惑。花时间去理解它可以帮助你写出更有扩展性和维护性的代码。
