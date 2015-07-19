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

List<Apple> box = ...;
Apple apple = box.get(0);

上面的代码表示：box 是一个 Apple 类型的列表引用，get 方法返回的 Apple 实例是不要求做类型转换的。
如果没有泛型，代码会是这样：

List box = ...;
Apple apple = (Apple) box.get(0);

不用说，泛型最主要的优点就是让编译器追踪参数类型，执行类型检查和类型转换：编译器保证类型转换不会失败。

如果依赖程序员去追踪对象类型和执行转换，那么运行时产生的错误将很难去定位和调试，然而有了泛型，编译器
可以帮助我们执行大量的类型检查，并且可以检测出更多的编译时错误。

### 泛型类和接口

一个类或者接口是泛型的意味着它有一个或多个类型变量。类型变量由尖括号分隔并遵循类(或接口)的名称：

public interface List<T> extends Collection<T> {
...
}

粗略来讲，类型变量扮演着参数和为编译器提供信息供其检查的角色。

 Java 库中的许多类，例如整个集合框架，都被修改为泛型了。例如我们第一个例子中的List类也是一个泛型类，在这个例子中，
 box 是List<Apple> 对象的引用。

 实际上，List 接口的get方法是：

 T get(int index);

get 方法返回一个T类型的对象，T是一个在List<T>中被声明的类型变量。

####泛型方法和构造函数

类似的，如果声明了一个或多个类型变量，方法和构造函数也可以是泛型的：

public static <t> T getFirst(List<T> list)

