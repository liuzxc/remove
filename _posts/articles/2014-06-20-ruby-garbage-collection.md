---
layout: post
title: 为什么ruby2.0的垃圾回收器让我们如此兴奋(译文)
comments: true
categories: articles
---

本文由 Jasonliu 翻译自 [Pat Shaughnessy](http://patshaughnessy.net/2012/3/23/why-you-should-be-excited-about-garbage-collection-in-ruby-2-0)。

<figure>
    <img src="/images/garbage-collection.jpg">
    <figcaption>位图标记垃圾回收器是一项令人激动且富有创造力的发明!</figcaption>
</figure>
{: .pull-left}

你也许听说过`Innokenty Mihailov`的`great Enumerable::Lazy feature`已经加入了ruby2.0
的代码，但是你也许没有听说过，一个更显著的变化在今年一月被合入了ruby2.0：一个被称作“位图标记”（bitmap marking）的垃圾回收算法。这个精致且具有创造性发明的开发者是Narihiro Nakamura,他从2008年开始就一直为此工作，在ruby1.9中已经包括了其实现的`lazy sweep`的垃圾回收算法。新的位图标记GC算法运行在web服务器的所有ruby进程中，可以显著的减少内存消耗。

但是位图标记的真正含义是什么呢？为什么会减少内存的消耗呢？如果你懂日语，你可以阅读`Narihiro Nakamura`
和`Yukihiro(Matz) Matsumotoa`在2008年出版的详细学术报告。我对其相当感兴趣，所以在本周
花了一些时间去学习ruby解释器的垃圾回收算法，这篇文章将会总结我所学到的东西。今天你不会学到任何与ruby
相关的编程技巧，但你可以更好的理解垃圾回收器是如何工作的，为什么ruby2.0是值得期待的，以及ruby内核开发人员是
多么的具有创造性！



#### Mark and Sweep

正如我在一月份写的一篇文章: [永远不要创建一个长度超过23的ruby字符串](http://patshaughnessy.net/2012/1/4/never-create-ruby-strings-longer-than-23-characters),每一个ruby字符串的值都是被MRI中的一个叫`RString`的C结构存储的，`RString`是`Ruby String`的简称。每一个`RString`结构被分割成了两等份，就像这样：

![RString](/images/rstring.png)

底部是真正的字符数据，顶部的单词`flags`表示ruby追踪与字符相关的不同类型的元数据值，它表示所有被使用的值是通过ruby程序存储在被称作`RArray`,`RHash`,`RFile`等类似的结构中。它们都有相同的基本样式：相同的数据和相同的标记集合。这种类型通常被称作`RValue`,即`Ruby Value`,它被跨内部对象类型所共享。

ruby在被称作`heaps`的数组中去分配和组织这些`RValue`结构。这里有一个ruby堆数组的概念图，它包含了三个字符串以及许多其他的`RValue`:

![heap(1)](/images/heap1.png)

一旦ruby程序运行，无论你什么时候创建变量或其他类型的值，ruby解释器都会在堆中找到一个可用的`RValue`结构，并且用它去存储这个值。当然，你完全没有必要担心什么，ruby解释器会自动的帮助你完成这个过程。

但是实际上，有时候并没有那么顺利。当`RValue`结构在堆中被消耗完的时候会发生什么呢？当没有地方去存储你程序需要的新值的时候会怎么样呢？这些比你想象的发生的更加频繁，因为有大量的的`RValue`结构在你无意识到的情况下被ruby所创建。实际上，你的ruby代码本身在解析和转化成字节代码的时候会生成大量的`RValue`结构。

当你的程序需要保存一个新值，但是又没有`RValue`结构可用的时候，ruby会运行垃圾回收器代码。垃圾回收器的工作就是去找到那些不再被程序所引用的`RValue`，这些`RValue`会被其他的值循环利用。下面就是它的工作原理，在一个高的级别上......

首先，GC代码“标记”所有活跃的`RValue`结构，也就是说，它循环的去遍历所有的变量和活跃的引用，这些是你程序必须拥有的`RValue`结构，并且每一个都会被打上一个称作`FL_MARK`的标记。

![fl-mark](/images/fl-mark.png)

这是ruby的`Mark and Sweep`垃圾回收算法的第一半，被标记的结构是活跃的并且是被程序一直使用的，它不会被释放和重用。

![sweeping](/images/sweeping.jpg)

一旦系统中所有像这样的结构被标记，剩下的结构会被移到一个单链表中，它使用`next`指针指向每一个`RValue`结构：在这个图中，我使用字母`M`表示堆数组中的`FL_MARK`标记，你看到的未被标记的`RValue`链表被称作空闲对象链表`free list`:

![free_list](/images/free-list.png)

正如你所猜到的一样，只要程序继续运行，空闲对象链表可以提供给新的`RValue`结构使用。现在，ruby程序在每一次分配新的对象或值的时候，都会使用空闲对象链表中的一个`RValue`，同时把它从链表中移除。最后，空闲对象链表会再一次为空，ruby会再次启动垃圾回收器。

一段时间之后，堆中也许已经没有未被标记的结构了，所有可用的`RValue`结构都被使用了。在这种情况下，ruby会分配一个完全新的堆去提供更多的`RValue`结构。（实际上，它会一次分配10个新堆）一个典型的ruby程序会有很多不同的堆数组。

#### Copy-On-Write: Unix如何跨不同的子进程共享内存

在我们了解位图标记算法`Bitmap marking`和它的重要性之前， 我们首先需要学习一个Linux和Unix，以及类Unix操作系统的新特性，它与内存管理和内存分配相关，即“写时拷贝优化”(`Copy-On-Write optimization`)。
在这些操作系统上，当一个进程调用`fork`去创建一个自拷贝的子进程时，新的子进程将会共享所有的内存，数据，变量等等，这些都是父进程之前被分配的，这就避免了不必要的内存拷贝，使`fork`调用更快速，也减少了总体的内存需求。

这就被称做“写时拷贝”（`Copy-On-Write`), 因为只有在需要且如果需要子进程去修改被共享的内存时，被共享的内存片段的不同拷贝才会被创建，这和ruby解释器用于管理`RString`值的原理相类似。对于细节，你可以参考我在一月份写的一篇文章：[Seeing double: how Ruby shares string values](http://patshaughnessy.net/2012/1/18/seeing-double-how-ruby-shares-string-values).

为了可以更好的理解，让我们看一下一个ruby进程的概念图：

![ruby-process](/images/ruby-process.png)

这里我们以拥有两个堆的ruby程序作为例子。现在假设一个ruby程序运行在一个web服务器上，也许它是一个rails web应用，现在来自于其他用户的第二个http请求到达了。

![sharing-memory](/images/sharing-memory.png)

现在我们有两个ruby进程正在运行。也许这个服务器正在运行Apache或者Passenger，它们派生一个单独的ruby进程去处理每个http请求。

在linux上，“写时拷贝优化”(`Copy-On-Write optimization`)的一个好处是：许多堆数组中的`RValue`结构在这两个ruby程序之间可以被共享，因为他们总是包含相同的值。乍看之下，这也许不是事实，为什么两个程序中的很多变量甚至任何变量都是相同的？请记住，在一个web服务器上，你实际上是运行了两个或者更多的相同代码的拷贝，并且一次又一次的创建相同的变量。并且在堆中的许多`RValue`结构实际上对应了你ruby程序本身的被解析的版本：静态语法树（`Abstract Syntax Tree`(AST)）的节点.只要每一个进程正在运行相同的代码，所有的这些节点都会有相同的值并且不会被改变。当然，有些数据值是不同的，它们将被单独保存在各自的进程中----例如用户在web表格中输入数据然后提交，在不同的记录上的sql查询结果等。

这听起来很棒，但是它无法在ruby中生效！

为什么不可以？因为只要ruby运行我上面讲到的`Mark & Sweep`垃圾回收算法，所有的这些静态树节点和许多其他堆中的`RValue`结构都将被标记，因为他们一直被ruby程序使用。这意味着它们被修改成了带有`FL_MARK`标记的结构，并且操作系统的写时拷贝代码不得不创建一个新的内存拷贝。所以在一个典型的ruby web服务器上，这才是真正发生的：

![not-sharing-memory](/images/not-sharing-memory.png)

一个小小的`FL_MARK`位是如此的具有破坏力！从实际发生的来看，它阻止了在正常情况下本可以极大地降低内存使用的可能。

> 这里值得注意的是：来自[Phusion](http://blog.phusion.nl/)的[Hongli Lai](http://izumi.plan99.net/blog/), 他是最流行的Passenger中间件的发明者，该组件连接Apache和基于ruby apps的Rac（ruby1.8的补丁），并且创建了知名的ruby新版本----[ruby企业版](http://www.rubyenterpriseedition.com/),他解决了这个问题并且做了大量的性能改进。事实上，这些年来，许多ruby1.8的应用通过使用REE获得了写时拷贝带来的优势。但是写时拷贝并不在标准的ruby1.8或者1.9中生效。

##### ruby2.0的垃圾回收器：位图标记（`Bitmap Marking`)

这里，我们迎来了`Narihiro Nakamura`针对ruby2.0的革新！通过代替给每一个`RValue`结构使用`FL_MARK`标记来表明ruby正在使用这个值还是应该释放它，ruby2.0保存着一些被称作“位图”（`Bitmap`)的信息。这里的`Bitmap`指的并不是一个图片文件，`Bitmap`这个概念引用自映射`RValue`结构的位集合：

![bitmap-marking](/images/bitmap-marking.png)

在ruby2.0中，针对每一个堆，都有一个与之相对应的内存结构，它包含了一系列的1或者0的位值。正如你所猜到的，1与在ruby1.8或者1.9进程中的`FL_MARK`标记是等同的，0与没有设置`FL_MARK`标记是等同的。换句话说，`FL_MARK`位从`RSting`和其他对象值结构中被移除，这种单独的内存区域被称作位图。

Narihiro实现位图是通过在每一个堆的开头添加一个header,它包含了一个指向位图的指针，以对应堆的`RValue`结构和一些其他的值。这意味着GC在处理`mark`部分的时候，ruby2.0可以标记正在使用的结构，实际上这并没有修改结构本身，并且允许Unix继续跨ruby进程的共享内存！当然，位图本身会被频繁的修改，只要使用位信号流，它们实际上是相当的小，并且可以被单独的存储在各自的进程中，不会消耗太多的内存。

一个有趣并且重要的细节是：堆的内存分配必须是”对齐的“(`aligned`)。当给堆分配内存的时候，与往常调用`malloc`不同的是：ruby的C代码调用`posix_memalign`，在Linux或者nix操作系统上，它返回一个新的内存，内存的两个地址边界是以2的幂对齐的。

这到底是什么意思？好吧，如果你熟悉C编程或者位运算，它允许ruby的C代码快速的计算出`header`结构的地址，`header`结构包含了指向位图的指针，以及一个给定的`RValue`对象的内存地址。让我们看一下ruby2.0的堆：

![memory-alignment](/images/memory-alignment.png)

假设ruby2.0的垃圾回收器代码需要标记这个堆中第15个`RValue`对象，通过引用`ptr`值。内存对齐的技巧可以帮助ruby2.0用`ptr`值快速的计算出这个堆的`header`结构的地址。所有的ruby2.0必须做的是清除`RValue`地址的最后几位，以“68”的16进制偏移为例，头结构的地址是`membase`或者`0x80FFC000`。

#####结论

乍看之下，垃圾回收并不是ruby中最富有魅力和最让人感兴趣的部分，但是如果你仔细的去研究它是如何工作的，你会发现有很多有趣的创新。实际上，位图标记算法的改变可以帮助ruby2.0的MRI在web服务器的生产环境下更好的工作，显著降低内存消耗，但是我发现位图算法对帮助rails运行的更好缺乏实质性的改善。研究ruby2.0下GC工作原理的最大乐趣是对复杂问题所产生的令人激动的，富有创造力的解决方案，我也希望那些才华出众的ruby内核开发人员的辛勤工作可以得到您的赞赏！