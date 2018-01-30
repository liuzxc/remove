---
layout: post
title: Java 提高篇(四) --- 异常处理
excerpt: java
categories: blog
comments: true
share: true
---

### 异常产生的时间和原因

异常分为运行时异常和编译时异常，异常产生的原因有很多，例如：打开不存在的文件，网络连接问题，
加载时类文件缺失等等。

### 错误和异常的区别

错误表明程序出现了很严重的问题或非正常的状况，应用不应该尝试去处理这样的错误。在正常环境下，程序不应该尝试去
捕捉错误，例如内存错误，硬件错误，JVM 错误等等。

异常表明了代码内部的状况，开发者应该处理这样的状况并采取必要的措施，例如：

* DivideByZero exception
* NullPointerException
* ArithmeticException
* ArrayIndexOutOfBoundsException

### 异常的类型

异常有两种类型：

1. 已检查异常
2. 未检查异常

#### 已检查异常

除了运行时异常以外的所有异常都被称作已检查异常，编译器会在编译期间去检查程序猿是否处理了它们。如果这些异常没有
被处理，将会出现编译错误。

已检查异常的例子：

* ClassNotFoundException
* IllegalAccessException
* NoSuchFieldException
* EOFException

#### 未检查异常

运行时异常也被称作为检查异常。编译器并不会检查是否有程序员处理这些异常，但程序员有处理这些异常的责任，并程序提供一个
安全的退出。

未检查异常的例子：

* ArithmeticException
* ArrayIndexOutOfBoundsException
* NullPointerException
* NegativeArraySizeException

异常层次结构：

<figure>
    <img src="/images/java04-1.png">
    <figcaption>Java 异常层次结构</figcaption>
</figure>
{: .pull-center}

### try...catch

catch block 总是跟着 try block 后面去处理其中的异常，try block 后面必须跟着一个 catch block 或 finally block。


#### 单个 catch block

```java
public class Example {
  public static void main(String[] args){
//    try{
//      int a = 0;
//      int b = 10 / a;
//      System.out.println("b: " + b);
//    } catch (ArithmeticException e){
//      System.out.println("Error: don't divide a number by zero");
//    }

    try{
      int a = 0;
      int b = 10 / a;
      System.out.println("b: " + b);
    } finally {
      System.out.println("Error: don't divide a number by zero");
      System.exit(0);
    }
  }
}
```


#### 多个 catch block

* 一个单独的 try block 可以有多个 catch 语句，但是每一个 catch block 只能定义一种异常类。
* catch(Exception e) 可以捕获所有类型的异常。
* 如果有多个 catch block， catch(Exception e) 最好放在最后。

```java
public class Example {
  public static void main(String[] args){
    try{
      int a[] = new int[5];
      for(int i=0;i<=5;i++){
        a[i] = 10 / (i+1);
      }
      System.out.println("No exception");
    } catch (ArithmeticException e) {
      System.out.println("Error: don't divide a number by zero");
    } catch (ArrayIndexOutOfBoundsException e){
      System.out.println("Warning: ArrayIndexOutOfBoundsException");
    } catch (Exception e) {
      System.out.println("Warning: Some Other exception");
    }
  }
}

Output:  Warning: ArrayIndexOutOfBoundsException
```


#### 嵌套 try ... catch

如果内层的 catch block 没有捕获到异常，那么控制权会转移到外层 catch block。

```java
public class Example {
  public static void main(String[] args){
    try{
        try{
          try{
            int a[] = new int[5];
            for(int i=0;i<=5;i++){
              a[i] = 10 / (i+1);
            }
            System.out.println("No exception");
            } catch (ArithmeticException e) {
            System.out.println("Error: don't divide a number by zero");
            }
           } catch (ArithmeticException e){
             System.out.println("Error: don't divide a number by zero");
           }
      } catch(ArrayIndexOutOfBoundsException e){
        System.out.println("Warning: ArrayIndexOutOfBoundsException");
      }
  }
}

Output => Warning: ArrayIndexOutOfBoundsException
```


### finally block

* 如果 finally block 出现异常，其行为和其他异常表现是一样的

```java
public class Example {
  public static void main(String[] args){
    try{
      try{
          int a = 1;
          int b = 10 / a;
          System.out.println("b: " + b);
        } finally {
        int c = 10 / 0;
          System.out.println("finally block");
          System.exit(0);
        }
    } catch (ArithmeticException e) {
      System.out.println("Error: don't divide a number by zero");
    }
  }
}

Output:

b: 10
Error: don't divide a number by zero
```


* 即使 try block 和 catch block 中包含了像 return，break，continue 这样的控制转换语句，
finally 中的代码依然会被执行。

```java
public class Example {
  static int test(){
    try{
      int a = 1;
      int b = 10 / a;
      System.out.println("b: " + b);
      return b;
    } finally {
      System.out.println("finally block");
    }
  }

  public static void main(String[] args){
    System.out.println(Example.test());
  }
}

Output:

b: 10
finally block
10
```


* 如果在 try block 中使用了 System.exit(0)，finally 块中的代码不会被执行。

```java
public class Example {
  static int test(){
      try{
          int a = 1;
          int b = 10 / a;
          System.out.println("b: " + b);
          System.exit(0);
          return b;
        } finally {
          System.out.println("finally block");
        }
  }

  public static void main(String[] args){
    System.out.println(Example.test());
  }
}

Output:

b: 10
```


### throw 关键字

Java 定义了一些异常类，例如 ArithmeticException， ArrayIndexOutOfBoundsException， NullPointerException
等等，这些异常发生的时候，JVM 会帮助我们抛出这些异常。但是如果我们想抛出自己定义的一些异常该怎么办呢？可以用 throw 关键字。

* 用户自己可以抛出一个系统已经定义的异常
* Java 中的异常都是 Throwable 类型，如果你尝试抛出一个非 Throwable 类型的异常，程序会报编译错误

#### throw 用户自定义异常

```java
class MyOwnException extends Exception{
  public MyOwnException(String msg){
    super(msg);
  }
}

public class Example {
  static int test(int length) throws MyOwnException{
    if(length < 0){
      throw new MyOwnException("length cannot be less than zero");
    } else{
      return length;
    }
  }

  public static void main(String[] args){
    try{
      Example.test(-1);
    } catch (MyOwnException e){
      System.out.println(e.getMessage());
    }
  }
}

Output:

length cannot be less than zero
```


#### throw 系统已定义异常

```java
public class Example {
  static int test(int length) {
    if(length < 0){
      throw new ArithmeticException("length cannot be less than zero");
    } else{
      return length;
    }
  }

  public static void main(String[] args){
    try{
      Example.test(-1);
    } catch (ArithmeticException e){
      System.out.println(e.getMessage());
    }
  }
}

Output:

length cannot be less than zero
```
