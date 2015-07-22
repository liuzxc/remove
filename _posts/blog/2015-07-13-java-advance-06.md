---
layout: post
title: Java 提高篇(六) --- 泛型（Generics）
excerpt:
categories: blog
comments: true
share: true
---

原文链接： http://www.javacodegeeks.com/2011/04/java-generics-quick-tutorial.html

### 引入泛型的动机是什么？

理解 Java 泛型最简单的方式就是把它当作一种语法糖，它可以帮助你做类型转换：

{% highlight java %}
List<Apple> box = ...;
Apple apple = box.get(0);
{% endhighlight %}

上面的代码表示：box 是一个 Apple 类型的列表引用，get 方法返回的 Apple 实例是不要求做类型转换的。
如果没有泛型，代码会是这样：

{% highlight java %}
List box = ...;
Apple apple = (Apple) box.get(0);
{% endhighlight %}

不用说，泛型最主要的优点就是让编译器追踪参数类型，执行类型检查和类型转换：编译器保证类型转换不会失败。

如果依赖程序员去追踪对象类型和执行转换，那么运行时产生的错误将很难去定位和调试，然而有了泛型，编译器
可以帮助我们执行大量的类型检查，并且可以检测出更多的编译时错误。

### 泛型类和接口

一个类或者接口是泛型的意味着它有一个或多个类型变量。类型变量由尖括号分隔并遵循类(或接口)的名称：

{% highlight java %}
public interface List<T> extends Collection<T> {
...
}
{% endhighlight %}

粗略来讲，类型变量扮演着参数和为编译器提供信息供其检查的角色。

Java库中的许多类，例如整个集合框架，都被修改为泛型了。例如我们第一个例子中的List类也是一个泛型类，在这个例子中，box 是List<Apple> 对象的引用。

实际上，List 接口的get方法是：

`T get(int index);`

get 方法返回一个T类型的对象，T是一个在List<T>中被声明的类型变量。

####泛型方法和构造函数

类似的，如果声明了一个或多个类型变量，方法和构造函数也可以是泛型的：

`public static <t> T getFirst(List<T> list)`

你可以在你自己的类或者Java库中的类发现泛型的优点：

#### 写时类型安全

例如下面一段代码，我们创建一个List<String>的实例并填充一些数据：

{% highlight java %}
List<String> str = new ArrayList<String>();
str.add("Hello ");
str.add("World.");
{% endhighlight %}

如果我们尝试添加一些其他类型的对象到 List<String>中，编译器将会抛出错误：

`str.add(1); // won't compile`

#### 迭代

标准库中的有些类，例如Iterator<T>，已经被扩展并实现了泛型，List<T>接口的iterator()方法返回了一个
Iterator<T>，可以方便的使用它而不需要做对象转换，并且通过T next()方法返回。

{% highlight java %}
import java.util.*;

public class TestIterator {

  public static void main(String[] args) {
    // TODO Auto-generated method stub
    List<String> str = new ArrayList<String>();
    str.add("liu");
    str.add("xing");
    str.add("qi");
    for(Iterator<String> iter = str.iterator(); iter.hasNext();){
      String s = iter.next();
      System.out.println(s);
    }
  }
}

Output:
liu
xing
qi
{% endhighlight %}

#### foreach

for each 语法也利用了泛型：

{% highlight java %}
for(String s: str){
  System.out.println(s);
}
{% endhighlight %}

相比之下，这个更容易阅读和维护。

#### 自动装箱和自动开箱（Autoboxing and Autounboxing）

Java 语言的自动装箱和自动开箱的特点是可以自动的使用和处理泛型，例如下面的代码：

{% highlight java %}
import java.util.*;

public class BoxingDemo {
  public static void main(String[] args){
    List<Integer> ints = new ArrayList<Integer>();
    ints.add(1);
    ints.add(2);
    int sum = 0;
    //自动将Integer转化为int
    for(int i: ints){
      sum+=i;
    }
    System.out.println(sum);
  }
}

Output: 3
{% endhighlight %}

需要注意的是，因为 boxing 和 unboxing 会带来性能损失，因此通常会有警告。