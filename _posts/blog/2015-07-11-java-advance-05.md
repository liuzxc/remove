---
layout: post
title: Java 提高篇(五) --- 集合类
excerpt: List, Set, Map
categories: blog
comments: true
share: true
---

* [List](#list)
* [Set](#set)
* [Map](#map)

### List
{: #list}

List 是一个有序集合（有时候也被称为序列）。List 可能包含有重复元素，元素可以被插入或者按照下标访问，并且索引以0开始。

* ArrayList
* LinkedList
* Vector

#### ArrayList

ArrayList 是一个数组队列，相当于动态数组。它由数组实现，随机访问效率高，随机插入、随机删除效率低。

ArrayList常见操作：

```java
import java.util.*;

public class ArrayListDemo {
  public static void main(String[] args){
    /* 数组初始化 */
    ArrayList<String> obj = new ArrayList<String>();
    obj.add("a1");
    obj.add("a2");
    obj.add("a3");
    obj.add("a4");
    obj.add("a5");

    /* 循环 */
    for(String element: obj){
      System.out.println(element);
    }

    /* 常见的一般操作 */
    System.out.println("Array list elements: " + obj);
    obj.add(1, "b1");
    System.out.println("Array list elements: " + obj);
    obj.add(5, "b5");
    System.out.println("Array list elements: " + obj);
    obj.remove("b1");
    obj.remove(4);
    System.out.println("Array list elements: " + obj);
    obj.set(1, "c1");
    System.out.println("Array list elements: " + obj);
    int pos = obj.indexOf("c1");
    System.out.println("c1 position is: " + pos);
    String str = obj.get(0);
    System.out.println("first position is: " + str);
    int size = obj.size();
    System.out.println("the size of array list is: " + size);
    boolean has = obj.contains("a5");
    System.out.println("the array list contains c5 ? : " + has);
    obj.clear();
    System.out.println("the array list is: " + obj);

    ArrayList<Integer> obj1 = new ArrayList<Integer>(Arrays.asList(1,4,2,8,6));

    /* 排序 */
    Collections.sort(obj1);
    System.out.println("Array list elements: " + obj1);

    Collections.sort(obj1, Collections.reverseOrder());
    System.out.println("Array list elements: " + obj1);
  }
}

Outputs:

a1
a2
a3
a4
a5
Array list elements: [a1, a2, a3, a4, a5]
Array list elements: [a1, b1, a2, a3, a4, a5]
Array list elements: [a1, b1, a2, a3, a4, b5, a5]
Array list elements: [a1, a2, a3, a4, a5]
Array list elements: [a1, c1, a3, a4, a5]
c1 position is: 1
first position is: a1
the size of array list is: 5
the array list contains c5 ? : true
the array list is: []
Array list elements: [1, 2, 4, 6, 8]
Array list elements: [8, 6, 4, 2, 1]
```

#### LinkedList

LinkedList 是一个双向链表。它也可以被当作堆栈、队列或双端队列进行操作。LinkedList随机访问效率低，但随机插入、随机删除效率高。

```java
import java.util.*;

public class LinkedListDemo {
  public static void main(String[] args){
    LinkedList<String> obj = new LinkedList<String>(Arrays.asList("a", "c", "b", "d", "e"));
    System.out.println("linkedlist elements are: " + obj);
    /* Inserts the element at the front of the list. */
    obj.push("f");
    System.out.println("linkedlist elements are: " + obj);
    /* Removes and returns the first element of the list. */
    obj.pop();
    System.out.println("linkedlist elements are: " + obj);
    Collections.sort(obj);
    String str = obj.get(3);
    System.out.println(str);
    for(String element: obj){
      System.out.println(element);
    }
  }
}

Output:

linkedlist elements are: [a, c, b, d, e]
linkedlist elements are: [f, a, c, b, d, e]
linkedlist elements are: [a, c, b, d, e]
d
a
b
c
d
e
```

#### Vector

Vector 是矢量队列，和ArrayList一样，它也是一个动态数组，由数组实现。但是ArrayList是非线程安全的，而Vector是线程安全的。
Vector 是同步的，这意味着它适用于线程安全的操作，但是在多线程环境下的性能很差。如果不需要线程安全的操作，建议用ArrayList代替
Vector (ArrayList 是非同步的，有更好的性能)。

```java
import java.util.*;

public class VectorDemo {

  public static void main(String[] args) {
    Vector<String> vec = new Vector<String>(2);
    vec.addElement("a");
    vec.addElement("b");
    vec.addElement("c");
    System.out.println("Size is: "+vec.size());
    System.out.println("Default capacity increment is: "+vec.capacity());
    for(String element: vec){
      System.out.println(element);
    }
  }

}

Output:

Size is: 3
Default capacity increment is: 4
a
b
c
```

### Set
{: #set}

Set 是不允许有重复记录的集合。Set 接口的实现主要有三种：

* HashSet
* LinkedHashSet
* TreeSet

#### HashSet

* HashSet 不允许存在重复值，如果添加了重复值。老的值将会被覆盖。
* HashSet 不保证迭代的顺序随时间保持不变，元素按随机顺序返回。
* HashSet 允许有 NUll 值。
* HashSet 是非同步的，但是可以显示的设置为同步的：Set s = Collections.synchronizedSet(new HashSet(...))

```java
import java.util.*;

public class HashSetDemo {

  public static void main(String[] args) {
    HashSet<String> hs1 = new HashSet<String>();
    hs1.add("d");
    hs1.add("b");
    hs1.add("a");
    hs1.add("c");
    System.out.println(hs1);
    hs1.add("e");
    System.out.println(hs1);

  }
}

Output:

[a, b, c, d]
[a, b, c, d, e]
```

#### TreeSet

* TreeSet 与 HashSet 很相似，但 HashSet 并不保证 Set 的迭代顺序，但 TreeSet 在遍历集合时按照
升序排序。所以对于 add，remove，contains，size等这样的操作，HashSet 比 TreeSet 有更好的性能，
HashSet 消耗的时间是常数级别的，而 TreeSet 是 log(n).

* TreeSet 是非同步的，但是也可以显示的指定为同步的：SortedSet s = Collections.synchronizedSortedSet(new TreeSet(...))

```java
import java.util.*;

public class TreeSetDemo {

  public static void main(String[] args) {
    TreeSet<String> ts = new TreeSet<String>();
    ts.add("desk");
    ts.add("apple");
    ts.add("black");
    ts.add("coffee");
    System.out.println(ts);
  }
}

Output: [apple, black, coffee, desk]
```

#### LinkedHashSet

LinkedHashSet 与 TreeSet 和 HashSet 很相似，除了以下几点不同：

1. hashSet 不保证 Set 的迭代顺序；
2. TreeSet 按照升序排序元素；
3. LinkedHashSet 会按照插入的顺序排序，输出的顺序与插入时的顺序相同；

```java
import java.util.*;

public class LinkedHashSetDemo {

  public static void main(String[] args) {
    LinkedHashSet<Integer> lhs = new LinkedHashSet<Integer>();
    lhs.add(10);
    lhs.add(1);
    lhs.add(5);
    lhs.add(0);
    System.out.println(lhs);
  }

}

Output: [10, 1, 5, 0]
```

### Map
{: #map}

Map 是一个对象，提供 key 到 value 的映射。map不能有重复的 key。Map 接口主要有
三种实现：

1. HashMap: 不保证迭代的顺序
2. TreeMap: 元素存储的红黑树当中，元素的书序基于它们的值。对于添加，删除，定位这样的操作，
TreeMap的性能比HashMap的差
3. LinkedHashMap: 元素的顺序基于它们插入时的顺序

#### HashMap

* 不保证迭代的顺序；
* 非同步的；
* 允许有null值和null键；

```java
import java.util.*;

public class HashMapDemo {

  public static void main(String[] args) {
    HashMap<String, Integer> hmap = new HashMap<String, Integer>();
    hmap.put("China", 100);
    hmap.put("Canada", 50);
        hmap.put("America", 80);
        hmap.put("Russan", 30);
        hmap.put(null, null);

        for(Map.Entry me: hmap.entrySet()){
          System.out.println("Key: "+ me.getKey() + " & Value: "+ me.getValue());
        }
  }
}

Output:

Key: null & Value: null
Key: Canada & Value: 50
Key: Russan & Value: 30
Key: China & Value: 100
Key: America & Value: 80
```

#### TreeMap

* TreeMap 是按照键值升序排列的
* 不允许键对象是null
* 是非同步的

```java
import java.util.*;
public class TreeMapDemo {

  public static void main(String[] args) {
    TreeMap<Integer, String> tm = new TreeMap<Integer, String>();
    tm.put(40, "China");
    tm.put(80, "America");
    tm.put(60, "Canada");
    tm.put(10, "Japan");
    for(Map.Entry me: tm.entrySet()){
      System.out.println("Key: "+ me.getKey() + " & Value: "+ me.getValue());
    }
  }
}

Output:

Key: 10 & Value: Japan
Key: 40 & Value: China
Key: 60 & Value: Canada
Key: 80 & Value: America
```


#### LinkedHashMap

元素的顺序基于其插入时的顺序

```java
import java.util.*;

public class LinkedHashMapDemo {

  public static void main(String[] args) {
    LinkedHashMap<Integer, String> lhm = new LinkedHashMap<Integer, String>();
    lhm.put(100, "China");
    lhm.put(10, "Japan");
    lhm.put(80,"America");
    lhm.put(30, "Canada");
    for(Map.Entry me: lhm.entrySet()){
      System.out.println("Key: "+ me.getKey() + " & Value: "+ me.getValue());
    }
  }
}

Key: 100 & Value: China
Key: 10 & Value: Japan
Key: 80 & Value: America
Key: 30 & Value: Canada
```