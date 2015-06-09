---
layout: post
title: 理解 ruby 中的 Blocks，Procs 和 Lambdas
excerpt: Blocks，Procs 和 Lambdas 是 ruby 最强有力的武器，也是最容易误解的
comments: true
share: true
categories: articles
---

原文链接：http://www.reactive.io/tips/2008/12/21/understanding-ruby-blocks-procs-and-lambdas/

* [Blocks](#blocks)
* [Procedures, AKA, Procs](#procedures)
* [Lambdas](#lambdas)
* [Method Objects](#method-objects)
* [Conclusion](#conclusion)

Blocks，Procs 和 Lambdas（在计算机科学中被称作`闭包`）是 Ruby 中最强大的方面之一，也是最容易让人误解的，
这可能是因为 Ruby 采用了多种方式来处理闭包。让事情变的复杂的原因是由于 Ruby 采用4种不同的方式来使用闭包，并且每一种都有所不同，甚至在某些时候使用闭包的方式看起来很荒谬。对于 Ruby 是如何使用闭包的，
有相当多的网站提供有用的信息，但是至今为止我一直都没有找到一个非常好的，希望本教程可以做到。

#### Blocks
{: #blocks}

在 Ruby 中，最通用，最容易，也是最毫无争议的 `ruby like` 的使用闭包的方式就是 blocks 。来看看我们很熟悉的语法：

{% highlight ruby linenos %}
array = [1, 2, 3, 4]

array.collect! do |n|
  n ** 2
end

puts array.inspect

# => [1, 4, 9, 16]
{% endhighlight %}

然而，这里发生了什么呢？

1. 首先，我们对数组调用了一个带有代码块的 collect! 方法；
2. collect! 方法的代码块中有一个变量 n，对其做平方运算；
3. 数组中的每一个元素都是原来的平方值；


在 collect! 方法内使用 blocks 是非常容易的，只需要想到 collect! 将对数组中的每一个元素使用代码块。
然而，如果我们想写一个自己的 collect! 方法要怎么办呢？它看起来会是什么样子？好吧，让我们生成一个 iterate! 的方法来看看：

{% highlight ruby linenos %}
class Array
  def iterate!
    self.each_with_index do |n, i|
      self[i] = yield(n)
    end
  end
end

array = [1, 2, 3, 4]

array.iterate! do |n|
  n ** 2
end

puts array.inspect

# => [1, 4, 9, 16]
{% endhighlight %}

一开始，我们重新打开了 Array 类并在其中添加了一个 iterate! 方法。我们保持了 Ruby 的命名规则并在方法名后加上叹号，目的是让
读者意识到这个方法也许很危险！接着我们就像使用 Ruby 内置方法 collect! 一样使用 iterate! 方法。

和属性不同，你不需要在方法中指定块的名字，反而你可以使用 yield 关键字，调用这个关键字可以执行块中的代码。注意我们是如何传递 n (
`each_with_index` 方法当前正在处理的整数)给 yield 的。传递给 yield 的属性对应了块中指定的变量列表。值被提供给块并通过 yield 返回。
让我们回顾一下发生了什么：

1. 对数组中的数字调用 iterate! 方法;
2. 当 yield 连同数字 n（第一次是1，第二次是2....)被调用，传递数字给块中的代码;
3. 块有数字提供（也叫做n）并对其平方，一旦最后一个值被块处理，它会自动返回；
4. Yield 输出通过块返回的值，然后重写数组的值；
5. 数组中的每个元素都按照这样的方式继续；

我们现有一个灵活的方式与方法进行交互，可以认为块是作为一个`API`提供给方法，你可以决定对数组中的每个元素进行平方，开方或者
把它们转化为字符串打印在屏幕上。yield 是执行块中代码的一种方式，还有另外一种方式，它被称作 `Proc`，让我们来看看。

{% highlight ruby linenos %}
class Array
  def iterate!(&code)
    self.each_with_index do |n, i|
      self[i] = code.call(n)
    end
  end
end

array = [1, 2, 3, 4]

array.iterate! do |n|
  n ** 2
end

puts array.inspect

# => [1, 4, 9, 16]
{% endhighlight %}

与之前的例子非常相似，然而却有两个不同之处。首先，我们传递了一个带 & 号的参数叫 &code，这个参数就是我们所说的块。
第二是在 iterate! 方法定义当中，使用关键字 call 来调用代码块，而不是 yield。结果是完全相同的。然而如果是这样，那
为什么要用不同的语法呢？好吧，我们再学习一点关于 block 到底是什么的相关内容，我们来看：

{% highlight ruby linenos %}
def what_am_i(&block)
  block.class
end

puts what_am_i {}

# => Proc
{% endhighlight %}

一个块就是一个 Proc ！这样说的话， Proc 又是什么？

#### Procedures, AKA, Procs
{: #procedures}

块非常的便利，而且语法简单，然而也许我们想要使用许多不同的块且使用多次，这样的话，一次又一次的传递相同的块将
成为重复的事情。Ruby 是完全面向对象的，通过把可重用的代码保存为一个对象，这样就可以把这种情况处理的相当简洁，
这种可重用的代码就被称作 Proc ( procedure 的简写). blocks 和 Procs 的唯一区别就是块是一个 Proc，但不能被保存，
也就是说块是一次性的解决方案。通过使用 Procs，我们可以这样做：

{% highlight ruby linenos %}
class Array
  def iterate!(code)
    self.each_with_index do |n, i|
      self[i] = code.call(n)
    end
  end
end

array_1 = [1, 2, 3, 4]
array_2 = [2, 3, 4, 5]

square = Proc.new do |n|
  n ** 2
end

array_1.iterate!(square)
array_2.iterate!(square)

puts array_1.inspect
puts array_2.inspect

# => [1, 4, 9, 16]
# => [4, 9, 16, 25]
{% endhighlight %}

>为什么 block 要小写而 Proc 要大写？
我总是把 Proc 大写是因为它是 Ruby 的一个类，然而 block 并不是类（毕竟它只是 Procs )而是 Ruby 的一种语法，正
因为这样，我才把 block 小写。在之后的教程当中，我也把 lambda 写成小写，也是因为这个原因。

请注意，我们并没有在 iterate! 方法前加上&号，这是因为传递 Procs 和传递其他数据类型一样，并没有什么不同。
正因为 Procs 可以被当作其他对象一样对待，我们可以做一些有趣的事情并推动 Ruby 解释器去做一些有趣的事情，让
我们试一下这个：

{% highlight ruby linenos %}
class Array
  def iterate!(code)
    self.each_with_index do |n, i|
      self[i] = code.call(n)
    end
  end
end

array = [1, 2, 3, 4]

array.iterate!(Proc.new do |n|
  n ** 2
end)

puts array.inspect

# => [1, 4, 9, 16]
{% endhighlight %}

上面是大多数语言处理闭包的方式，并且和调用一个块的效果是一样的。如果你觉得这不像`ruby like`，我会
表示赞同。如果事实是这样，为什么不只使用 blocks ？答案很简单，假如我们想传递两个或更多的块到方法中会怎么样？
如果事实是这样，blocks 将很快的具有局限性。然而如果通过 Procs。我们可以这样做：

{% highlight ruby linenos %}
def callbacks(procs)
  procs[:starting].call

  puts "Still going"

  procs[:finishing].call
end

callbacks(:starting => Proc.new { puts "Starting" },
          :finishing => Proc.new { puts "Finishing" })

# => Starting
# => Still going
# => Finishing
{% endhighlight %}

所以，什么时候你应该使用 blocks 而不是 Procs？我的逻辑是这样的：

1. Block: 你的方法是把一个对象分解成更小的片段，你想让你的用户与这些片段交互；
2. Block: 你想要自动的运行多个表达式，就像数据库迁移；
3. Proc: 你想要重用一个代码块多次；
4. Proc: 你的方法有一个或多个回调；

#### Lambdas
{: #lambdas}

到目前为止，你已经用两种方式来使用 Procs：当做属性直接传递和保存作为一个变量。这些 Procs 扮演的角色和
其他语言调用匿名函数或 lambda 是很相似的。更有趣的是，Ruby 也提供 lambdas。让我们看看：

{% highlight ruby linenos %}
class Array
  def iterate!(code)
    self.each_with_index do |n, i|
      self[i] = code.call(n)
    end
  end
end

array = [1, 2, 3, 4]

array.iterate!(lambda { |n| n ** 2 })

puts array.inspect

# => [1, 4, 9, 16]
{% endhighlight %}

乍看之下，lambdas 与 Procs 极其相似，然而它们有两个细微的不同之处：第一个不同之处是，不像 Porcs，
lambdas 会检查传递参数的个数。

{% highlight ruby linenos %}
def args(code)
  one, two = 1, 2
  code.call(one, two)
end

args(Proc.new{|a, b, c| puts "Give me a #{a} and a #{b} and a #{c.class}"})

args(lambda{|a, b, c| puts "Give me a #{a} and a #{b} and a #{c.class}"})

# => Give me a 1 and a 2 and a NilClass
# *.rb:8: ArgumentError: wrong number of arguments (2 for 3) (ArgumentError)
{% endhighlight %}

我们看到，对于 Proc 这个例子，多余的参数被设置为 nil ，然而 lambdas 却抛出了异常。

第二个不同之处是 lambdas 有返回。这就意味着 Proc return 将从方法中退出并返回当前值，lambdas 将返回其只到方法中并让
方法继续运行。感到疑惑么？让我们看一个例子：

{% highlight ruby linenos %}
def proc_return
  Proc.new { return "Proc.new"}.call
  return "proc_return method finished"
end

def lambda_return
  lambda { return "lambda" }.call
  return "lambda_return method finished"
end

puts proc_return
puts lambda_return

# => Proc.new
# => lambda_return method finished
{% endhighlight %}

在 proc_return 方法中，方法遇到 return 关键字，停止执行接下来的方法，然后返回字符串 `Proc.new`。
另一方面，lambda_return 方法遇到 lambda，它返回一个字符串 `lambda`，然后继续运行知道碰到下一个 return，
然后输出`lambda_return method finished`。为什么会有不同？

答案是：在概念上，procedures 和 method 有所不同。Ruby 中的 Proc 的行为是代码片段而不是方法。因为这个，Proc return
就是 proc_return 方法的 return ，并作出相同的反应。然而 lambda 的行为与方法类似，它们检查参数个数并且不会覆盖被
调用方法的 return。基于这个原因，理解 lambdas 的最好方式是把它当作一个匿名方法。

那么，在什么时候应该写一个匿名方法（lambdas）而不是 Proc 呢？下面的代码展示了一个这样的案例：

{% highlight ruby linenos %}
def generic_return(code)
  code.call
  return "generic_return method finished"
end

puts generic_return(Proc.new { return "Proc.new" })
puts generic_return(lambda { return "lambda" })

# => *.rb:6: unexpected return (LocalJumpError)
# => generic_return method finished
{% endhighlight %}

部分的 Ruby 语法是参数（本例中参数是一个 Proc）中是不能有 return 关键字的。然而，lambda 的行为就像一个方法，
它有一个字面上的 return，因此它可以满足这个需求。不同的语义下会有不同的情况，就像下面这个例子：

{% highlight ruby linenos %}
def generic_return(code)
  one, two    = 1, 2
  three, four = code.call(one, two)
  return "Give me a #{three} and a #{four}"
end

puts generic_return(lambda { |x, y| return x + 2, y + 2 })

puts generic_return(Proc.new { |x, y| return x + 2, y + 2 })

puts generic_return(Proc.new { |x, y| x + 2; y + 2 })

puts generic_return(Proc.new { |x, y| [x + 2, y + 2] })

# => Give me a 3 and a 4
# => *.rb:9: unexpected return (LocalJumpError)
# => Give me a 4 and a
# => Give me a 3 and a 4
{% endhighlight %}

这里，generic_return 方法期望通过闭包返回两个值，通过一个 lambda，一切都很简单！然而如果使用 Proc，
我们最终不得不利用 Ruby 是如何赋值数组的。

那么，什么时候应该使用 Proc 而不是 lambdas？反之亦然。老实说，除了参数检查以外，不同之处就是你如何看待
闭包。如果你想留在传递代码块的心态上，请使用 Proc。如果发送一个方法到另一个方法，然后返回一个方法对你
是有意义的，请使用 lambdas。但是如果 lambdas 只是对象形式中的方法，我们可以存储已经存在的方法，并且可以像
 Procs 一样传递他们么？对于这个，Ruby 有它的秘技。

#### Method Objects
{: #method-objects}

你已经有一个有效的方法，但是你想把它作为一个闭包传递给另外一个方法，并且保持你的代码 `DRY`。为了做到这个，
你可以利用 Ruby 的 method 方法。

{% highlight ruby linenos %}
class Array
  def iterate!(code)
    self.each_with_index do |n, i|
      self[i] = code.call(n)
    end
  end
end

def square(n)
  n ** 2
end

array = [1, 2, 3, 4]

array.iterate!(method(:square))

puts array.inspect

# => [1, 4, 9, 16]
{% endhighlight %}

在这个例子中，我们已经有一个叫 square 的方法可以很好的解决手头上的任务。正因为如此，通过把它
转化为一个方法对象，我们可以把它作为一个参数重用它并传递它到 iterate! 方法。这是一个新的对象
类型么？

{% highlight ruby linenos %}
def square(n)
  n ** 2
end

puts method(:square).class

# => Method
{% endhighlight %}

如你所料，square 不是一个 Proc，而是一个 method。这个 method object 的行为像一个 lambda，因为概念是相同的，只不过
这是个命名方法（叫做 square），而 lambda 是匿名方法。

#### Conclusion
{: #conclusion}

重述一下，我们认识了四个闭包类型：blocks，Procs，lambdas 和 Methods。我们也知道了 blocks 和 Procs 的行为像
代码片段，而 lambdas 和 Methods 的行为像方法。最后，通过一些代码示例，你可以知道什么时候，使用哪个以及如何有效的
使用它们。现在，你应该可以开始在你的代码中使用 Ruby 这个有趣的特性，并且可以提供灵活而有效的方法给其他的开发人员使用。
