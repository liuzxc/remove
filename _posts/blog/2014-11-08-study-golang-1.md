---
layout: post
title: go语言学习笔记（一）
excerpt:
modified:
categories: blog
---

###Go的基本类型

* bool //默认值为false
* string //默认值为空字符串
* int  int8  int16  int32  int64 //默认值为0
* uint uint8 uint16 uint32 uint64 uintptr //默认值为0
* byte // uint8 的别名
* rune // int32 的别名
* float32 float64 //默认值为0
* complex64 complex128 //默认值为0

###类型声名和转换

与C不同的是Go在不同类型之间的变量赋值时需要显式转换, go语言中只能且仅能同类型之间进行运算。 表达式`T(v)`将值`v`转换为类型`T`。
{% highlight go %}
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
var a, b, c int = 0, 1, 2 //左右的数量要匹配，否则会报错

i := 42
f := float64(i)
u := uint(f)
{% endhighlight %}

如果想把数字转化为字符串要怎么办呢？

在ruby语言中，将数字转化为字符串只需要`to_s`就可以，但是go语言需要导入特定的包`strconv`。

{% highlight go %}
package main

import (
	"fmt"
	"strconv"
)

func main() {
	x := 10
	y := float64(x)
	fmt.Println("string:" + toString(y))
}

func toString(a interface{}) string {

	if value, ok := a.(int); ok {
		return strconv.Itoa(value)
	}
	if value, ok := a.(int8); ok {
		return strconv.Itoa(int(value))
	}
	if value, ok := a.(int16); ok {
		return strconv.Itoa(int(value))
	}
	if value, ok := a.(int32); ok {
		return strconv.Itoa(int(value))
	}
	if value, ok := a.(int64); ok {
		return strconv.Itoa(int(value))
	}
	if value, ok := a.(uint); ok {
		return strconv.Itoa(int(value))
	}
	if value, ok := a.(float32); ok {
		return strconv.FormatFloat(float64(value), 'f', 5, 32)
	}
	if value, ok := a.(float64); ok {
		return strconv.FormatFloat(value, 'f', 5, 64)
	}
	return "wrong type"
}
{% endhighlight %}

> 
Go语言里面有一个语法，可以直接判断是否是该类型的变量： value, ok = element.(T)，这里value就是变量的值，ok是一个bool类型，element是interface变量，T是断言的类型。
如果element里面确实存储了T类型的数值，那么ok返回true，否则返回false。

###数组

类型 [n]T 是一个有 n 个类型为 T 的值的数组。

{% highlight go %}
var a [10]int //定义变量 a 是一个有十个整数的数组
{% endhighlight %}

###数组切片slice

一个 slice 会指向一个序列的值，并且包含了长度信息。[]T 是一个元素类型为 T 的 slice。
创建数组切片的方法主要有两种:

####基于数组

数组切片可以基于一个已存在的数组创建。数组切片可以只使用数组的一部分元素或者整个数组来创建,甚至可以创建一个比所基于的数组还要大的数组切片.
{% highlight go %}
// 先定义一个数组
var myArray [10]int = [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
// 基于数组创建一个数组切片
var mySlice []int = myArray[:5]
{% endhighlight %}

####直接创建

并非一定要事先准备一个数组才能创建数组切片。Go语言提供的内置函数make()可以用于
灵活地创建数组切片。
{% highlight go %}
//创建一个初始元素个数为5的数组切片,元素初始值为0:
mySlice1 := make([]int, 5)
//创建一个初始元素个数为5的数组切片,元素初始值为0,并预留10个元素的存储空间:
mySlice2 := make([]int, 5, 10)
//使用append()函数为slice新增元素
mySlice = append(mySlice2, 1, 2, 3)
//使用append()函数将一个数组切片追加到另一个数组切片的末尾
mySlice2 = append(mySlice2, mySlice1...) //加上省略号相当于把mySlice1包含的所有元素打散后传入
//直接创建并初始化包含5个元素的数组切片:
mySlice3 := []int{1, 2, 3, 4, 5}
{% endhighlight %}

###range

使用range关键字可以让遍历代码显得更整洁。range表达式有两个返回值,第一个是索引,第二个是元素的值:
{% highlight go %}
for i, v := range mySlice {
	fmt.Println("mySlice[", i, "] =", v)
}
{% endhighlight %}

###map

map是一堆键值对的未排序集合.
{% highlight go %}
//myMap是声明的map变量名,string是键的类型,value则是其中所存放的值类型。
var myMap map[string] value
//可以使用Go语言内置的函数make()来创建一个新map
myMap = make(map[string] value)
//也可以选择是否在创建时指定该map的初始存储能力
myMap = make(map[string] value, 100)
//删除map中的元素
delete(myMap, key)
{% endhighlight %}