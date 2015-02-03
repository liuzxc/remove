---
layout: post
title: Rails App中重复记录产生的原因
modified:
categories: blog
---

在我工作的rails项目中，经常发现数据库中存在重复的记录，重复的记录会导致用户获取不到准确的数据，也会让数据库产生
很多脏数据。产生重复记录的原因有两个方面：

* 在应用层没有作出限制，在创建记录之前没有做任何校验；
* 数据库设计不合理，未作唯一性约束；

##Uniqueness Validation

在rails应用中，通过添加唯一性验证（Uniqueness validation）可以帮助我们在应用层阻止重复记录的产生。

如果我有一张user_porudct的用户产品表，我希望每一个用户的产品ID都是唯一的，可以通过以下方法来实现:

{% highlight ruby %}
class UserProduct < ActiveRecord::Base
  validates_uniqueness_of :product_id, scope: :user_id
  ......
end
{% endhighlight %}

在数据存入数据库之前，它会检查当前用户的product_id是否已经存在，如果存在则会引发异常。

这种方式是rails中最常规的防止重复记录产生的手段，它在一定程度上确实其作用，但是不幸的是，它并不能防止由于竞争
条件产生的重复记录。

> 官方文档对此做了描述 [validates_uniqueness_of](http://api.rubyonrails.org/classes/ActiveRecord/Validations/ClassMethods.html#method-i-validates_uniqueness_of)

##Race Condition

> 多个线程或者进程在读写一个共享数据时结果依赖于它们执行的相对时间，这种情形叫做竞争。
竞争条件发生在当多个进程或者线程在读写数据时，其最终的的结果依赖于多个进程的指令执行顺序。

| id     | user_id | product_id | amount | created_at        | updated_at         |
|--------|---------|------------|--------|-------------------|--------------------|
|181115  |811115   |1800        |2       |2014-10-16 03:00:13|2014-10-16 03:00:13 |
|181116  |811115   |1800        |2       |2014-10-16 03:00:13|2014-10-16 03:00:13 |
{: rules="groups"}

从上面两条记录可以看出，在同一时间，一个玩家产生了两条product_id相同的记录，虽然我在model层添加了唯一性验证，
但是还是无法保证记录唯一，这就是由于竞争条件引起的记录重复问题。在同一时间点，用户点击了两次产品添加按钮，两个请求同时
到达服务器，它们顺利的通过了rails的唯一性验证，因为在这时，记录还没有被插入数据库。而在数据库，这两个请求的执行顺序
可能会是这样的：

<figure>
  <img src="/images/duplicate_01.png" alt="image">
</figure>

重复的记录就这样产生了。

##Fighting Race Condition

防止重复记录被创建的方法非常简单，通过创建唯一索引，从数据库的级别防止重复记录产生。在index migration之前，需要将数据库中
已经存在的重复记录删除，否则migration会出错。

{% highlight ruby %}
add_index  :user_product, [:product_id, :user_id],  unique: true
{% endhighlight %}

<figure>
  <img src="/images/duplicate_02.png" alt="image">
</figure>
