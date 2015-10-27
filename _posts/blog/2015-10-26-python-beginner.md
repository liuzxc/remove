---
layout: post
title: Python 初探
excerpt: 主要探寻一下 Python 和 Ruby 的一些不同之处
categories: blog
comments: true
share: true
---

最近离职换了新的工作，开始使用 Python，说起来我和 Python 还比较有缘，大四实习接触的第一门语言就是 Python，可惜时间短，也仅限于自己写点小东西，没有在实际项目中应用过。这几天一直抽空看 python，用惯了ruby的我，学习python也比较快，基本上不会纠结在语法上，感觉它们相似点多于不同点。回想找工作的时候，招python的公司拒绝我这样的ruby工程师，眼光会不会太狭隘了点，哈哈😄

本篇文章不会在语法层面上做太多的解释，毕竟这样的教程网上一片，主要还是理解一些python中有，而ruby中没有的，或者实现不同的某些点。

#### if __name__ == '__main__':

在很多的python脚本中都可以看到这样的声明：`if __name__ == '__main__':`，它到底是什么意思呢？

在 C/C++/Java 这样的语言中，都有入口函数/main函数的感念，python 也有main函数，只是实现机制有些不同。Python 使用缩进来组织代码，除了类定义和函数定义外，对于没有缩进的代码，Python 都会去执行他们，这些代码可以被理解为 main 函数。如果多个文件都含有没有缩进的代码，那么 python 如何判断应该哪个是主执行文件呢？

所以 python 引入了__name__ 属性，当文件被调用时，__name__ 为文件名，当文件被执行时，__name__ 为 __main__。

{% highlight python %}
# name.py
if __name__ == '__main__':
  print "name.py is executed"
else:
  print "name.py is imported"

print __name__
{% endhighlight %}

当执行 name.py 的时候，输出为：

{% highlight python %}
name.py is executed
__main__
{% endhighlight %}

当 import name.py 的时候：

{% highlight python %}
test.py is imported
test
{% endhighlight %}

> ruby 并没有main()函数的概念，它会逐行执行文件中的所有代码

#### list comprehension（列表推导式）

有一次做python面试题的时候，第一道题就是用 list comprehension 取列表中的偶数，当时不知道 list comprehension是什么东东，写出的代码是这个样子：

{% highlight python %}
def func(list):
  temp = []
  for x in list:
    if x % 2 == 0: temp.append(x)
  return temp
{% endhighlight %}

实际上应该是这个样子的：

{% highlight python %}
[x for x in list if x % 2 == 0]
{% endhighlight %}

由此可见，列表推导式的最大作用就是使代码变的更简洁。

嵌套循环：

{% highlight python %}
x=[1, 2, 3, 4]
y=[5, 6, 7, 8]
[i * j for i in x for j in y]
=> [5, 6, 7, 8, 10, 12, 14, 16, 15, 18, 21, 24, 20, 24, 28, 32]
{% endhighlight %}

字典操作：

{% highlight python %}
d = {"a": 100, "b": 200, "c": 300}
[x + str(y) for x, y in d.iteritems()]
=> ['a100', 'c300', 'b200']
{% endhighlight %}

> ruby 没有列表推导式，但是 map 方法可以达到类似的效果

#### Higher-order function 高阶函数

所谓高阶函数，就是函数的参数可以是其他函数。例如：

{% highlight python %}
def sum(a, b):
  return a + b

#函数可以赋值给变量
f = sum

def func(x, y, f):
  return f(x, y)

func(1, 2, sum) => 3
{% endhighlight %}

##### Python 中常用的内置高阶函数

1. map(function, iterable, ...)

{% highlight python %}
map(sum, [1,2,3,4],[5,6,7,8]) => [6, 8, 10, 12]
map(lambda x: x**2, [1,2,3,4,5]) => [1, 4, 9, 16, 25]
map(lambda x,y: x+y, [1,2,3,4,5],[1,2,3,4,5]) => [2, 4, 6, 8, 10]
{% endhighlight %}

2. filter(function, iterable)

{% highlight python %}
filter(lambda x: x % 2 == 0, [1,2,3,4,5,6]) => [2, 4, 6]
{% endhighlight %}

3. reduce(function, iterable[, initializer])

{% highlight python %}
reduce(lambda x, y: x + y, [1,2,3,4,5,6]) => 21
{% endhighlight %}

> ruby 和 python 都有 lambda，lambda 可以作为匿名函数使用


