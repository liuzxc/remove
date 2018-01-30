---
layout: post
title: Java 提高篇(六) --- 泛型（Generics）
excerpt: java
categories: blog
comments: true
share: true
---

原文链接： http://www.javacodegeeks.com/2011/04/java-generics-quick-tutorial.html

### 引入泛型的动机是什么？

理解 Java 泛型最简单的方式就是把它当作一种语法糖，它可以帮助你做类型转换：

```java
List<Apple> box = ...;
Apple apple = box.get(0);
```

上面的代码表示：box 是一个 Apple 类型的列表引用，get 方法返回的 Apple 实例是不要求做类型转换的。
如果没有泛型，代码会是这样：

```java
List box = ...;
Apple apple = (Apple) box.get(0);
```

不用说，泛型最主要的优点就是让编译器追踪参数类型，执行类型检查和类型转换：编译器保证类型转换不会失败。

如果依赖程序员去追踪对象类型和执行转换，那么运行时产生的错误将很难去定位和调试，然而有了泛型，编译器
可以帮助我们执行大量的类型检查，并且可以检测出更多的编译时错误。

### 泛型类和接口

一个类或者接口是泛型的意味着它有一个或多个类型变量。类型变量由尖括号分隔并遵循类(或接口)的名称：

```java
public interface List<T> extends Collection<T> {
...
}
```

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

```java
List<String> str = new ArrayList<String>();
str.add("Hello ");
str.add("World.");
```

如果我们尝试添加一些其他类型的对象到 List<String>中，编译器将会抛出错误：

`str.add(1); // won't compile`

#### 迭代

标准库中的有些类，例如Iterator<T>，已经被扩展并实现了泛型，List<T>接口的iterator()方法返回了一个
Iterator<T>，可以方便的使用它而不需要做对象转换，并且通过T next()方法返回。

```java
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
```

#### foreach

for each 语法也利用了泛型：

```java
for(String s: str){
  System.out.println(s);
}
```

相比之下，这个更容易阅读和维护。

#### 自动装箱和自动开箱（Autoboxing and Autounboxing）

Java 语言的自动装箱和自动开箱的特点是可以自动的使用和处理泛型，例如下面的代码：

```java
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
```

需要注意的是，因为 boxing 和 unboxing 会带来性能损失，因此通常会有警告。

#### 子类型

Java作为一种面向对象的语言，可以构建层次结构的类型：

在Java里，T类型的字类型可以是T的扩展也可以是T接口的实现。由于自类型是一种可传递的关系，如果A是B的子类，B是C的子类，
那么A也会是C的子类。如上图所示：

* 富士苹果是苹果的字类型
* 苹果是水果的字类型
* 富士苹果也是水果的字类型

每一个 Java类型都是Object的子类型。每一个类型B的子类型A都可以被赋值给B的引用：

Apple a = ...;
Fruit f = a;

#### 泛型的子类型

如果一个🍎实例的引用被赋值给了一个Fruit的引用，那么List<Apple> 和 List<Fruit>之间是什么关系？
谁是谁的子类型，更普遍的说法是，如果类型A是类型B的子类型，那么C<A>和C<B>的关系是什么？

令人惊讶的是，答案是: 没有关系。用更正式的话说，泛型类型的子类型之间关系是不变的。

这意味着下面的代码是不合法的：

List<Apple> apples = ...;
List<Fruit> fruits = apples;

下面这个也不合法：

List<Apple> apples;
List<Fruit> fruits = ...;
apples = fruits;

但是为何会这样呢？难道苹果不是水果么？一盒苹果也不是一盒水果么？在某种程度上讲，答案是肯定的，但是
类型（或者类）把状态和行为封装在了一起，如果一盒苹果是一盒水果会发生什么情况呢？

List<Apple> apples = ...;
List<Fruit> fruits = apples;
fruits.add(new Strawberry());

如果是的话，我们可以添加其他类型的水果到列表当中，但这一定会被禁止的。有一种更直观的方式是：一盒
水果并不是一盒苹果，因为它可以包含其他类型的水果，比如🍓。

#### 这真的是个问题么？

这不应该是，令Java程序员感到惊讶的是数组和泛型的行为时矛盾的。后者的子类型关系是不变的，而前者的
子类型关系却是协变的：如果A的是B的子类型，那么A[]也是B[]的子类型。

Apple[] apples = ...;
Fruit[] fruits = apples;

> 协变是什么意思？
> 维基百科中是这样定义的：在一门程序设计语言的类型系统中，一个类型规则或者类型构造器是：
> * 协变（covariant），如果它保持了子类型序关系≦。该序关系是：子类型≦基类型。
> * 逆变（contravariant），如果它逆转了子类型序关系。
> * 不变（invariant），如果上述两种均不适用。
> 首先考虑数组类型构造器： 从Animal类型，可以得到Animal[]（“animal数组”）。 是否可以把它当作
> * 协变：一个Cat[]也是一个Animal[]
> * 逆变：一个Animal[]也是一个Cat[]
> * 以上二者均不是（不变)

但是等等！也许我们可以把草莓添加到苹果数组当中：

Apple[] apples = new Apple[1];
Fruit[] fruits = apples;
fruits[0] = new Strawberry();

代码确实可以编译通过，但是会抛出一个ArrayStoreException的运行时错误。因为对于数字的存储操作，
Java 运行时会去检查类型的兼容性，你应该意识到这会有性能损失。

泛型可以安全的使用和改正Java数组的的类型安全弱点。