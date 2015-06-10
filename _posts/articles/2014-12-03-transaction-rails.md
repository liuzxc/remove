---
layout: post
title: Rails 中的事务
excerpt: 使用事务过程中常见的错误和解决方案
comments: true
share: true
categories: articles
---

本文由 Jasonliu 翻译自 [Mark Daggett's Blog](http://markdaggett.com/blog/2011/12/01/transactions-in-rails/)。

最近一段时间，我的任务是为已存在的项目写事务测试，这给了我更多学习代码和提高测试覆盖率的机会。一般情况下，大部分事务相关的代码看起来都没有问题，但是有些却被误用，或使用的不够高效，我认为这是由于对事务在rails中工作原理的误解导致的， 并且我认为我应该花一些时间描述一下我发现的最常见的错误和使用事务的经验。

这篇文章中的大部分例子都不是我自己写的，它们来源于rails源码中的示例。

[http://api.rubyonrails.org/classes/ActiveRecord/Transactions/ClassMethods.html](http://api.rubyonrails.org/classes/ActiveRecord/Transactions/ClassMethods.html)

#### 使用事务的原因

我们使用事务把SQL语句包裹起来,以确保数据库的更改只发生在所有操作均成功的情况下。事务帮助开发者确保应用数据的完整性。最典型的例子是银行交易，现金被存入然后取出，如果其中一步失败，整个交易过程应当被重置。这个例子可以用以下代码来描述：

{% highlight ruby linenos %}
ActiveRecord::Base.transaction do
  david.withdrawal(100)
  mary.deposit(100)
end
{% endhighlight %}

在rails中，事务作为类或实例方法可以被所有的`ActiveRecord model`使用，所以这样写也是合法的：

{% highlight ruby linenos %}
Client.transaction do
  @client.users.create!
  @user.clients(true).first.destroy!
  Product.first.destroy!
end

@client.transaction do
  @client.users.create!
  @user.clients(true).first.destroy!
  Product.first.destroy!
end
{% endhighlight %}

你可能注意到了，这几个事务示例都引用了不同的model类。在一个单独的事务代码块中混合model类是毫无问题的，因为事务是被绑定到数据库连接而不是model实例。通常情况下，数据库中的多个记录更新必须作为一个单元被成功执行的时候才需要事务。并且rails已经将`save`和`destroy`方法放在了事务当中，所以更新一个单独的记录是不需要事务的。

#### 事务回滚触发器

事务通过一个叫rollback的进程去重置记录的状态。在rails中，`rollback`只会被异常触发，这是了解事务的一个关键点。在一些项目中，我发现了有些事务代码块根本就不会回滚，因为其中的代码不会抛出异常。在这里，我对之前的银行示例代码稍微做了修改：

{% highlight ruby linenos %}
ctiveRecord::Base.transaction do
  david.update_attribute(:amount, david.amount -100)
  mary.update_attribute(:amount, 100)
end
{% endhighlight %}

rails中，当更新操作失败后，`update_attribute`不会抛出异常，它会返回`false`，正因为如此，所以你应该确保方法在遇到失败的时候抛出异常。更好的做法是：

{% highlight ruby linenos %}
ActiveRecord::Base.transaction do
  david.update_attributes!(:amount => -100)
  mary.update_attributes!(:amount => 100)
end
{% endhighlight %}

> 注意：在rails中，［!］表示方法失败时会抛出异常。

我也看到过一些这样的例子：事务中使用`find_by`查找记录，当没有找到记录的时候，`find_by`会返回nil，而`find`方法则会抛出一个`ActiveRecord::RecordNotFound`的异常。

通常情况下，抛出一个异常会导致事务回滚并且异常会被传递出去，但是如果你抛出一个`ActiveRecord::Rollback`异常，事务会回滚，但是异常不会被传递出去。


#### 使用嵌套事务

在编程的时候，一个常见的错误是误用和过度使用嵌套事务。当你在一个事务中嵌套另外一个事务时，子事务会成为父事务的一部分，这会导致意想不到的结果，让我们看看rails API 文档中的例子：

{% highlight ruby linenos %}
User.transaction do
  User.create(:username => 'Kotori')
  User.transaction do
    User.create(:username => 'Nemu')
    raise ActiveRecord::Rollback
  end
end
{% endhighlight %}

正如之前提到的，异常`ActiveRecord::Rollback` 并不会传播到事务之外，以至于父事务并不会收到子事务中抛出的异常，因此子事务和父事务中的记录都被创建了。可能这样想会简单一点，嵌套事务就像子事务的内容被移到父事务中去了，子事务中的内容为空。

为了确保rollback被父事务接收到，你必须添加`:requires_new => true`选项到子事务。再次使用rails源码中的例子，你可以像这样触发嵌套事务回滚：

{% highlight ruby linenos %}
User.transaction do
  User.create(:username => 'Kotori')
  User.transaction(:requires_new => true) do
    User.create(:username => 'Nemu')
    raise ActiveRecord::Rollback
  end
end
{% endhighlight %}

一个事务只取决于当前的数据库连接，如果你的应用需要一次写多个数据库，你需要把方法放入嵌套事务当中。例如：

{% highlight ruby linenos %}
Client.transaction do
  Product.transaction do
    product.buy(@quantity)
    client.update_attributes!(:sales_count => @sales_count + 1)
  end
end
{% endhighlight %}

#### 围绕事务的回调

正如之前所说，`save`和`destroy`已经被放入了事务当中，这意味着像`after_save`这样的回调函数也是事务的一部分，也会被回滚。因此，如果你有一些在事务之外需要执行的代码，可以使用像`after_commit`或`after_rollback`这样的回调函数。

#### 事务陷进

在事务中，不要去捕获`ActiveRecord::RecordInvalid`这样的异常，因为对于`Postgres`这样的数据库，这种异常会导致事务失效。一旦事务失效，为了保证工作正确，你必须重启事务。

#### 避开常见的反模式

1. 当只有单独的记录被更新的时候使用事务

2. 无必要的嵌套事务

3. 包含不会回滚的代码的事务

4. 在controller中使用事务

> 作者在此并没有解释为什么不要在`controller`中使用事务。我认为是完全可以在`controller`中使用事务的，这和在`model`中使用事务的效果是一样的。但是rails是标准的`MVC`架构，在`controller`中使用事务违反了`MVC`架构的设计原则：控制层不应该直接和数据库打交道，与数据库的交互应该交给`model`层来完成。