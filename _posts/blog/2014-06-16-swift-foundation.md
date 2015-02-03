---
layout: post
title: swift基础部分
categories: blog
---

苹果最近推出了新一代的编程语言swift，听说学习门槛降低不少，之前一直想尝试一下IOS
开发，但是由于OC的语法确实让我难以接受，尤其是使用了一年多的ruby之后，更难以接受
OC这样的老古董了，便想尝试学习一下swift。

迫不及待的把mac升级到10.9.3,同时下载了Xcode6-beta版,找了一堆干货资料准备开干。
先看了[51CTO的视频资料](http://edu.51cto.com/lesson/id-26620.html)，比较
初级，但是讲的比较细。github上早已经有了[swift教程中文版](http://numbbbbb.github.io/the-swift-programming-language-in-chinese/)


#基础部分

###常量和变量

* 声明常量： let maximumNumberOfLoginAttempts = 10 
* 声明变量： var currentLoginAttempt = 0

###注释

* //            :单行注释
* /\*.....\*/   :多行注释

###数值型类型转换
SomeType(ofInitialValue)

###类型别名
typealias AudioSample = UInt16 ：AudioSample是UInt16的别名

###元组
元组（tuples）把多个值组合成一个复合值。元组内的值可以使任意类型，并不要求是相同类型。

{% highlight c linenos %}
let http404Error = (404, "Not Found") // http404Error 的类型是 (Int, String)
你可以将一个元组的内容分解（decompose）成单独的常量和变量
let (statusCode, statusMessage) = http404Error
// statusCode = 404, statusMessage = "Not Found"/如果你只需要一部分元组值，分解的时候可以把要忽略的部分用下划线（_）标记：
let (justTheStatusCode, _) = http404Error; justTheStatusCode =404
此外，你还可以通过下标来访问元组中的单个元素，下标从零开始：
http404Error.0 = 404; http404Error.1 = "Not Found"
你可以在定义元组的时候给单个元素命名：
let http200Status = (statusCode: 200, description: "OK")
给元组中的元素命名后，你可以通过名字来获取这些元素的值：
http200Status.statusCode = 200; http200Status.description = "OK"
{% endhighlight %}
作为函数返回值时，元组非常有用。

###可选类型
可选类型（optionals）用来处理值可能缺失的情况。我开始对这个概念特别的困惑，因为中文版的swift手册完全没把这个概念讲解清楚，看了慕课网的一个教程之后才理解可选类型的作用，强烈推荐看一下[可选类型optionals](http://www.imooc.com/video/1936).
一个可选类型的值表示可能有值，也可能没有值，没有值的时候就是nil。
ruby中的nil是一个对象，但是swift的nil是一个确切的值，用来表示值缺失。
> 注意：nil不能用于非可选的常量和变量。如果你的代码中有常量或者变量需要处理值缺失的情况，请把它们声明成对应的可选类型。

###数组（Array）
swift的数组和ruby的很相似，基本的用法差别不大，如果你熟悉ruby，你很快就可以明白swift数组的用法。

swift数组构造语句： 
{% highlight c linenos %}
var shoppingList: String[] = ["Eggs", "Milk"]
var shoppingList = ["Eggs", "Milk"]
var someInts = Int[]() #构造一个空数组
var threeDoubles = Double[](count: 3, repeatedValue:0.0)
count：数组元素数量
repeatedValue： 数组元素的初始值
可以不用显示的声明shoppingList的类型值。
{% endhighlight %}

###字典（Dictionary）
swift的字典相当于ruby中的hash。
{% highlight c linenos %}
var airports: Dictionary<String, String> = ["TYO": "Tokyo", "DUB": "Dublin"]
var airports = ["TYO": "Tokyo", "DUB": "Dublin"]
var namesOfIntegers = Dictionary<Int, String>() #创建一个空字典
{% endhighlight %}

###控制流
swift提供两种for循环

1.for-in 循环
和ruby的for-in循环几乎是一模一样的;
{% highlight c linenos %}
for element in array {
    println("....")
}
{% endhighlight %}
2.For条件递增,和C语言的for循环类似;
{% highlight c linenos %}
for var index = 0; index < 3; ++index {
    println("...")
}
{% endhighlight %}

3.while和do-while均与C语言的类似;

4.switch语句：和ruby的switch语句很类似，均不需要显示的break;

###控制转移语句
1. continue语句告诉一个循环体立刻停止本次循环迭代，重新开始下次循环迭代.
2. break语句会立刻结束整个控制流的执行.
