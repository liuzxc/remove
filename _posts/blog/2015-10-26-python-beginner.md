---
layout: post
title: Python 初探
excerpt: 寻找 python 和 ruby 的不同之处
categories: blog
comments: true
share: true
---

最近离职换了新的工作，开始使用 python，说起来我和 python 还比较有缘，大四实习接触的第一门语言就是 python，可惜时间短，也仅限于自己写点小东西，没有在实际项目中应用过。这几天一直抽空看 python，用惯了 ruby 的我，学习 python 也比较快，基本上不会纠结在语法上，但是还是感觉它们的不同点还是非常多。

本篇文章不会在语法层面上做太多的解释，毕竟这样的教程网上一片，主要还是理解一些 python 中有，而 ruby 中没有的，或者实现不同的某些点。

#### `if __name__ == '__main__':`

在很多的 python 脚本中都可以看到这样的声明：`if __name__ == '__main__':`，它到底是什么意思呢？

在 C/C++/Java 这样的语言中，都有入口函数/main 函数的感念，python 也有 main 函数，只是实现机制有些不同。python 使用缩进来组织代码，除了类定义和函数定义外，对于没有缩进的代码，python 都会去执行他们，这些代码可以被理解为 main 函数。如果多个文件都含有没有缩进的代码，那么 python 如何判断应该哪个是主执行文件呢？

所以 python 引入了 `__name__` 属性，当文件被调用时，`__name__` 为文件名，当文件被执行时，`__name__` 为 `__main__`。

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

有一次做 python 面试题的时候，第一道题就是用 list comprehension 取列表中的偶数，当时不知道 list comprehension 是什么东东，写出的代码是这个样子：

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

##### python 中常用的内置高阶函数

**map(function, iterable, ...)**

{% highlight python %}
map(sum, [1,2,3,4],[5,6,7,8]) => [6, 8, 10, 12]
map(lambda x: x**2, [1,2,3,4,5]) => [1, 4, 9, 16, 25]
map(lambda x,y: x+y, [1,2,3,4,5],[1,2,3,4,5]) => [2, 4, 6, 8, 10]
{% endhighlight %}

**filter(function, iterable)**

{% highlight python %}
filter(lambda x: x % 2 == 0, [1,2,3,4,5,6]) => [2, 4, 6]
{% endhighlight %}

**reduce(function, iterable[, initializer])**

{% highlight python %}
reduce(lambda x, y: x + y, [1,2,3,4,5,6]) => 21
{% endhighlight %}


> ruby 和 python 都有 lambda，lambda 可以作为匿名函数使用

#### 装饰器（decorator）

装饰器是 python 中比较有趣的功能，它可以在不改变原有方法的基础上，为方法添加新的功能。为了熟悉这个概念，我花了不少时间去了解，我发现网上的很多例子都显得太过复杂难懂，对于新手来说，从简单的例子开始会更好理解一点。

{% highlight python %}
#首先写一个原始方法，它简单的输出一个句子
def original_function():
  print "This is a original function"

#然后我们添加一个装饰方法
def decorate_original_function(func):
  def wapper_original_function():
    print "do something before original function runs:"
    func()
    print "do something after original function runs:"
  return wapper_original_function

func = decorate_original_function(original_function)
func()

output:
do something before original function runs:
This is a original function
do something after original function runs:
{% endhighlight %}

从这个例子我们可以看出装饰器的运行原理：把原函数(original_function)作为参数(func)传递给装饰器函数(decorate_original_function)，然后返回一个函数(wapper_original_function)，你可以在返回函数中添加你想要的功能（比如此处的两个print），既可以在 func() 之前，也可以在 func() 之后。这样就会理解为什么 pyhton 中装饰器的应用如此之广泛。

使用 python 的装饰器语法：

{% highlight python %}
# decorate_original_function(original_function)
@decorate_original_function
def original_function():
  print "This is a original function"

original_function()

output:
do something before original function runs:
This is a original function
do something after original function runs:
{% endhighlight %}

> `@decorate_original_function` 是 `decorate_original_function(original_function)` 的简写

#### 方法和访问修饰符

##### 方法

在 ruby 中，一个方法是实例方法还是类方法是通过判断方法当前的所属对象决定的。

{% highlight ruby %}
class MyClas
  def test; end  #实例方法
  def self.test1; end #类方法
end
{% endhighlight %}

而 python 是把当前对象作为参数传递给方法，从而判断方法是实例方法还是类方法。python 中还有一种方法叫静态方法，从 ruby 的角度来讲，类方法和静态方法是一样的，但 python 不同，它的静态方法不会以实例或类最为第一个参数，但是实例对象和类都可以调用静态方法，我到现在并不清楚 python 中静态方法的作用到底是什么，网上的说法是 **经常有一些跟类有关系的功能但在运行时又不需要实例和类参与的情况下需要用到静态方法**。

{% highlight python %}
class MyClass:
  def test(self): #实例方法
    pass

  @classmethod
  def test1(cls): #类方法
    pass

  @staticmethod
  def test2(): #静态方法
    pass
{% endhighlight %}

写个简单的例子看下：

{% highlight python %}
class MyClass(object):
  def test(self):
    print "this is a instance method"

  @classmethod
  def test1(cls):
    print "this is a class method"

  @staticmethod
  def test2():
    print "this is a static method"

class MyClass1(MyClass):
  pass


print "MyClass:"

m = MyClass()
m.test()

m.test1()
MyClass.test1()

m.test2
MyClass.test2()

print "MyClass1:"

m1 = MyClass1()
m1.test()
m1.test1()
m1.test2()

output:

MyClass:
this is a instance method
this is a class method
this is a class method
this is a static method
MyClass1:
this is a instance method
this is a class method
this is a static method
{% endhighlight %}

从上面的程序中我们可以看出 python 语言的一些特点：

1. 实例方法只能被类的实例所调用（ruby 说我和你一样）
2. 类方法除了可以被类本身调用，还可以被实例对象调用（ruby 表示做不到）
3. 静态方法可以被类和实例对象调用（ruby 表示我没有你这样的静态方法)

##### 访问修饰符

令我感到惊讶的是 python 竟然没有访问修饰符，Java 和 ruby 表示你丫太前卫，我们跟不上。python 通过一种约定（convention）来实现访问权限控制。变量和方法名前面不加下划线表示它们是 public 的，
加一个下划线表示 protected, 加两个表示 private。

通过一个例子来了解一下：

{% highlight python %}
class MyClass(object):
  _name = "liuzxc"
  __age = 26

  def test(self):
    print "this is a instance method"

  def _test(self):
    self.__test() #私有方法只有在本类中可以调用
    print "this is a protected instance method"

  def __test(self):
    print "this is a private instance method"

class MyClass1(MyClass):
  pass

print "MyClass:"

m = MyClass()
m.test()
m._test()
# m.__test()  #调用失败

print "MyClass1:"

m1 = MyClass1()
m1.test()
m1._test()
# m1.__test() #调用失败
print MyClass1._name
# print MyClass1.__age #调用失败

output:

MyClass:
this is a instance method
this is a private instance method
this is a protected instance method
MyClass1:
this is a instance method
this is a private instance method
this is a protected instance method
liuzxc
{% endhighlight %}