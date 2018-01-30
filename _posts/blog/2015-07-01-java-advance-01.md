---
layout: post
title: Java 提高篇(一) --- Java 关键字
excerpt: static, abstract, final
categories: blog
comments: true
share: true
---

* [Static](#static)
* [Abstact](#abstact)
* [Final](#final)

部分内容引用自：http://www.cnblogs.com/chenssy/p/3386721.html

### 一、 Static
{: #static}

Java 中 static 表示“全局”或者“静态”的意思，用来修饰成员变量和成员方法，也可以修饰代码块。 Java 把内存分为栈内存和堆内存，其中栈内存用来存放一些基本类型的变量、数组和对象的引用，堆内存主要存放一些对象。在 JVM 加载一个类的时候，若该类存在 static 修饰的成员变量和成员方法，则会为这些成员变量和成员方法在固定的位置开辟一个固定大小的内存区域，有了这些“固定”的特性，那么 JVM 就可以非常方便地访问他们。

同时被 static 修饰的成员变量和成员方法是独立于该类的，它不依赖于某个特定的实例变量，也就是说它被该类的所有实例共享。所有实例的引用都指向同一个地方，任何一个实例对其的修改都会导致其他实例的变化。

#### 1. 静态变量

static修饰的变量我们称之为静态变量，没有用static修饰的变量称之为实例变量，他们两者的区别是：

* 静态变量是随着类加载时被完成初始化的，它在内存中仅有一个，且JVM也只会为它分配一次内存，同时类所有的实例都共享静态变量，可以直接通过类名来访问它。
* 但是实例变量则不同，它是伴随着实例的，每创建一个实例就会产生一个实例变量，它与该实例同生共死。

```java
public class Test {
  // 静态变量和实例变量都有默认值，局部变量没有默认值
  private static int i;
  private int j;

  public Test(){
    i++;
    j++;
  }

  public static void main(String[] args){
    Test t1 = new Test();
    Test t2 = new Test();
    System.out.println("t1 i is: " + Test.i);
    System.out.println("t2 i is: " + Test.i);
    System.out.println("t1 j is: " + t1.j);
    System.out.println("t2 j is: " + t2.j);
  }
}

output:

t1 i is: 2
t2 i is: 2
t1 j is: 1
t2 j is: 1
```

#### 2. 静态方法

static 修饰的方法我们称之为静态方法，我们通过类名对其进行直接调用。由于他在类加载的时候就存在了，它不依赖于任何实例，所以 static 方法必须实现，也就是说他不能是抽象方法 abstract。

```java
public class Test {
	private static int i;
	private int j;

	public Test(){
		i++;
		j++;
	}

	static void test(){
		i = i + 10;
		System.out.println("i is: " + i);
	}

    void test1(){
		j = j + 10;
		System.out.println("j is: " + j);
	}


	public static void main(String[] args){
		Test t1 = new Test();
		Test t2 = new Test();
		Test.test();
		Test.test();
		t1.test1();
		t2.test1();
	}
}

i is: 12
i is: 22
j is: 11
j is: 11
```

需要注意的几点是：

* 静态方法不能调用非静态变量和非静态方法，而非静态方法可以调用静态方法和静态变量；
* 静态方法和非静态方法不能同名；

### 二、 Abstact
{: #abstact}

#### 1. 抽象类

被声明为abstract的被称为抽象类，抽象类有几个特性：

1. 它可以含有抽象方法，也可以没有抽象方法，但有抽象方法的类一定是抽象类；
2. 抽象类不可以被实例化，但可以被继承；
3. 抽象类可以有静态变量和静态方法

#### 2. 抽象方法

抽象方法没有具体的实现（没有花括号，后面只跟一个分号），如果一个抽象类被继承，那么子类必须实现父类的抽象方法。

```java
abstract class Test {

    abstract void test();

    void test1(){
    	System.out.println("intsance method");
    }
}

class Test1 extends Test{
	void test(){
		System.out.println("implement sbatact methd");
	}

	public static void main(String[] args){
		Test1 t = new Test1();
		t.test();
		t.test1();
	}
}

output:

implement sbatact methd
intsance method
```

### 二、 Final
{: #final}

#### 1. final variable

final variable 就是一个常量，一旦被初始化就不可以被改变。

```java
class Test1 {
  final double PI = 3.14; //常量的名称最好大写

  public Test1(){
    PI = 3.14;
  }

    void test(){
      System.out.println("PI is: " + PI);
    }

  public static void main(String[] args){
    Test1 t = new Test1();
    t.test();
  }
}

PI is: 3.14
```

##### Blank final variable

在声明时未初始化的 final variable 被称作 blank final variable， blank final variable必须在
构造函数中被初始化，否则会抛出编译错误。

```java
class Test1 {
	final double PI;

	Test1(){
		PI = 3.14; //在构造函数中初始化
	}

    void test(){
    	System.out.println("PI is: " + PI);
    }

	public static void main(String[] args){
		Test1 t = new Test1();
		t.test();
	}
}

PI is: 3.14
```

##### Uninitialized static final variable

在声明阶段未初始化的 static final variable 只能在静态代码块中被初始化

```java
class Test1 {
	static final double PI;

	static {
		PI = 3.14;
	}

    void test(){
    	System.out.println("PI is: " + PI);
    }

	public static void main(String[] args){
		Test1 t = new Test1();
		t.test();
	}
}

PI is: 3.14
```

#### 2. final method

final method 不能被覆盖。也就是说子类可以调用父类的 fianl method，但是不能覆盖它。

```java
class Test {
	static final double PI = 3.14;

    final void test(){
    	System.out.println("PI is: " + PI);
    }
}

class Test1 extends Test{

	public static void main(String[] args){
		Test1 t = new Test1();
		t.test();
	}
}

PI is: 3.14
```

#### 3. final class

final calss 不能被继承

```java
final class Test1 {
	static final double PI = 3.14;

    final void test(){
    	System.out.println("PI is: " + PI);
    }

	public static void main(String[] args){
		Test1 t = new Test1();
		t.test();
	}
}

PI is: 3.14
```
