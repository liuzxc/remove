---
layout: post
title: go语言学习笔记（二）
excerpt:
modified:
categories: blog
---

流程控制

#### if语句

{% highlight go %}
if a > b {
	fmt.Println(...)
}
//如果出现多条件判断，需要使用`&&`, `||`, 和ruby不同的是，golang不支持`and`, `or`
if a == b && b == c {
	fmt.Println(...)
}
if a == b || b == c {
	fmt.Println(...)
}
{% endhighlight %}

#### switch语句

golang的switch语句和ruby的截然不同：

{% highlight go %}
package main

import "fmt"

func main() {
	var bonus float32 = 0.0
	var I float32 = 0.0
	fmt.Print("输入利润：")
	fmt.Scanf("%f\n", &I)
	switch {
	case I > 1000000: //表达式后面必须要有`:`
		bonus = (I - 1000000) * 0.01
		I = 1000000
		fallthrough //继续执行紧跟的下一个case, ruby不支持这样做
	case I > 600000:
		bonus += (I - 600000) * 0.015
		I = 600000
		fallthrough
	case I > 400000:
		bonus += (I - 400000) * 0.03
		I = 400000
		fallthrough
	case I > 200000:
		bonus += (I - 200000) * 0.05
		I = 200000
		fallthrough
	case I > 100000:
		bonus += (I - 100000) * 0.075
		I = 100000
		fallthrough
	default: //类似于ruby的`else`
		bonus += I * 0.1
	}
	fmt.Println(bonus)
}
{% endhighlight %}


#### 循环语句

golang只支持`for`循环，不支持`while`

{% highlight go %}
//类似于C语言的循环
for i := 0; i < count; i++ {	
}
//无限循环
i := 0
for {
	.....
	i++
	if i > 100 {
		break
	}
}
{% endhighlight %}

#### 三元表达式

golang没有三元表达式，不过可以自己通过函数模仿

{% highlight go %}
import "fmt"

type B bool

func main() {
	fmt.Println(B(80 >= 90).F("A", "B"))
}

func (b B) F(x, y interface{}) interface{} {
	if bool(b) == true {
		return x
	}
	return y
}
{% endhighlight %}


