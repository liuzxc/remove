---
layout: post
title: Ruby 中一些好用的方法
excerpt: 总结了自己在工作中和 stackoverflow 上学到的一些很有用的方法
categories: blog
comments: true
share: true
---

#####inject

`inject`是我使用最频繁的方法了，它的强大之处在于可以方便的对嵌套的数组，哈希等混合数据结构进行合并或求和，
可以有效减少代码量。

例如最常见的数组套哈希：

{% highlight ruby linenos %}
array = [{a:100}, {b:200}, {c:300}]

array.inject(0) { |sum, e| sum += e.values.first } #600
array.inject({}) { |sum, e| sum.merge e } #{:a=>100, :b=>200, :c=>300}

{% endhighlight %}

* inject():括号中的是sum的初始值
* sum, e: 和在前，数组元素在后，中间必须以逗号隔开

#####group_by

`group_by`适用于对于数组和hash的分组. 在`stackoverflow`,我经常遇到这样的问题：

`array`根据相同的`school_id`进行分组

{% highlight ruby linenos %}
array =  [{"school_id"=>"1",
		  "plan_type"=>"All",
		  "view"=>"true",
		  "create"=>"true",
		  "approve"=>"true",
		  "grant"=>"true",
		  "region_id"=>nil},
		 {"school_id"=>"1",
		  "plan_type"=>"All",
		   "edit"=>"true",
		   "region_id"=>nil},
		 {"school_id"=>"2",
		  "plan_type"=>"All",
		  "edit"=>"true",
		  "grant"=>"true",
		  "region_id"=>nil}]

array.group_by { |e| e["school_id"] }
=> {"1"=>[{"school_id"=>"1", "plan_type"=>"All", "view"=>"true", "create"=>"true", "approve"=>"true", "grant"=>"true", "region_id"=>nil}, {"school_id"=>"1", "plan_type"=>"All", "edit"=>"true", "region_id"=>nil}], "2"=>[{"school_id"=>"2", "plan_type"=>"All", "edit"=>"true", "grant"=>"true", "region_id"=>nil}]}
{% endhighlight %}

多条件分组：

{% highlight ruby linenos %}
array.group_by { |e| [e["school_id"], e["plan_type"]] } #将多个条件放在数组当中
=> {["1", "All"]=>[{"school_id"=>"1", "plan_type"=>"All", "view"=>"true", "create"=>"true", "approve"=>"true", "grant"=>"true", "region_id"=>nil}, {"school_id"=>"1", "plan_type"=>"All", "edit"=>"true", "region_id"=>nil}], ["2", "All"]=>[{"school_id"=>"2", "plan_type"=>"All", "edit"=>"true", "grant"=>"true", "region_id"=>nil}]}
{% endhighlight %}

####reduce

`reduce`作用和`inject`优点类似，但是它比`inject`还要简洁

{% highlight ruby linenos %}
(5..10).inject {|sum, n| sum + n }  # 45
(5..10).reduce(:+)  # 45
(5..10).reduce(1, :+)  # 46 (括号中第一个参数是初始值，第二个是方法名)
{% endhighlight %}

以`group_by`中的`array`为例，将相同`school_id`的hash进行合并

{% highlight ruby linenos %}
array.group_by { |e| e["school_id"] }.values.map { |i| i.inject({}) { |sum ,e| sum.merge e }}
=> [{"school_id"=>"1", "plan_type"=>"All", "view"=>"true", "create"=>"true", "approve"=>"true", "grant"=>"true", "region_id"=>nil, "edit"=>"true"}, {"school_id"=>"2", "plan_type"=>"All", "edit"=>"true", "grant"=>"true", "region_id"=>nil}]
{% endhighlight %}

可以使用`inject`将hash合并，但是使用reduce效果会更好

{% highlight ruby linenos %}
array.group_by { |e| e["school_id"] }.values.map { |i| i.reduce(:merge) }
=> [{"school_id"=>"1", "plan_type"=>"All", "view"=>"true", "create"=>"true", "approve"=>"true", "grant"=>"true", "region_id"=>nil, "edit"=>"true"}, {"school_id"=>"2", "plan_type"=>"All", "edit"=>"true", "grant"=>"true", "region_id"=>nil}]
{% endhighlight %}

####zip

`zip`可以将两个数组合并为一个二维数组

{% highlight ruby linenos %}
a= [1,2,3,4,5]
b=[6,7,8,9,10]
a.zip(b)
=> [[1, 6], [2, 7], [3, 8], [4, 9], [5, 10]]
{% endhighlight %}

如果`a.length > b.length`，b中缺少的以nil代替 ，

{% highlight ruby linenos %}
a=[1,2,3,4,5,6]
b=[6,7,8,9,10]
a.zip(b)
=> [[1, 6], [2, 7], [3, 8], [4, 9], [5, 10], [6, nil]]
{% endhighlight %}

如果`a.length < b.length`，b中多余的直接被丢弃

{% highlight ruby linenos %}
a= [1,2,3,4]
b=[6,7,8,9,10]
a.zip(b)
=> [[1, 6], [2, 7], [3, 8], [4, 9]]
{% endhighlight %}