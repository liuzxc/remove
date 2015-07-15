---
layout: post
title: Java 提高篇(三) --- 多态
excerpt: 方法覆盖和方法重载
categories: blog
comments: true
share: true
---

### 什么是多态？

多态可以让方法基于对象做不同的事情。换句话说，多态允许你定义一个接口并且有多个实现。听起来有些
困惑，别担心，我们会从细节上进行讨论。

* 多态是一种功能，允许一个接口用于普通类的操作；
* 一个操作在不同的实例上可能会展现出不同的行为；
* 行为取决于操作数据的类型；
* 允许对象有不同的内部结构，共享相同的外部接口；
* 多态被广泛的用于实现继承；

下面两个概念展示了 Java 不同的多态类型。

1. 方法重载
2. 方法覆盖

###方法重载（Method Overloading）

* 如果在 Java 中调用一个重载方法，必须使用不同类型或（和）数量的参数去决定哪一个重载方法被调用；
* 重载方法也许有不同的返回类型，返回类型本身并不足以区别两个版本的方法；

{% highlight java %}
	public class Overload {
		void test(){
			System.out.println("no params");
		}
	  //无法通过编译，因为只根据返回类型无法区分这两个方法, 必须根据参数的类型和个数
		int test(){
			System.out.println("return a int");
			return 1;
		}
	}
{% endhighlight %}

* 当 Java 要调用一个重载方法的时候，它只简单的执行参数匹配的那个方法；

{% highlight java %}
public class Overload {
	void test(){
		System.out.println("no params");
	}

	int test(int a){
		System.out.println("has one param");
		return 1;
	}

	int test(int a, int b){
		System.out.println("has two params");
		return a + b;
	}

	public static void main(String[] args){
		Overload obj = new Overload();
		obj.test();
		obj.test(1);
		obj.test(1, 2);
	}
}

Output:

no params
has one param
has two params
{% endhighlight %}

* 允许用户实现编译时多态；
* 一个重载方法可以抛出不同的异常；
* 可以有不同的访问修饰符；

方法重载的规则：

1. 重载可以发生在同一个类中或其子类；
2. 构造函数可以被重载；
3. 重载方法必须要有不同的参数列表；
4. 重载方法有相同的名称，但是参数不同；
5. 参数可以是类型不同，数量不同，或者二者皆不同；
6. 重载方法可能有相同或不同的返回类型；
7. 方法重载也被称作编译时多态；

### 方法覆盖（Method Overriding)

子类于基类的方法相同。

方法覆盖的原则：

* 仅适用于继承的方法；
* 在运行时，对象类型决定了哪一个覆盖方法被调用；
* 覆盖方法可以有不同的返回类型；

		需要注意的是：

		* 如果返回类型是 void 或主数据类型，覆盖方法的返回类型必须保持一致；
		* 如果返回类型是引用类型（reference type)，那么可以不同，但必须兼容（如下所示）；

{% highlight java %}
class Pet{
	Pet(){
		System.out.println("Pet");
	}
}

class Dog extends Pet{
	Dog(){
		System.out.println("Dog");
	}
}

class Overrid1 {
  Pet test(){
	  return new Pet();
  }
}

class Overrid2 extends Overrid1{
	Dog test(){
		return new Dog();
	}
}

class Overrid{
	public static void main(String[] args){
		Overrid2 obj = new Overrid2();
		obj.test();
	}
}
{% endhighlight %}

* 覆盖方法不能有更严格的访问修饰符；

{% highlight java %}
class Overrid1 {
  int test(){
	  return 1;
  }
}

class Overrid2 extends Overrid1{
  //因为父类的test方法的访问权限是默认的，所以子类的访问权限必须不小于父类，即默认的，protected 或 public
	public int test(){
		return 2;
	}
}

class Overrid{
	public static void main(String[] args){
		Overrid2 obj = new Overrid2();
		obj.test();
	}
}
{% endhighlight %}

* 抽象方法必须被覆盖（抽象方法的作用就是让子类的方法覆盖）；
* static 和 final 方法不能被覆盖；

>为什么静态方法不能被覆盖？覆盖依赖于类的实例，而静态方法和类实例并没有什么关系。而且静态方法在编译时就已经确定，而方法覆盖是在运行时确定的。

* 构造函数不能被覆盖；
* 方法覆盖也被称作运行时多态；
* 使用 super 关键字调用父类被覆盖的方法；