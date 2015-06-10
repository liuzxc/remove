---
layout: post
title: Ruby 实现各种排序查找算法
excerpt:
comments: true
categories: blog
share: true
---

之前在微博上看到旧金山大学的David Galles教授用HTML5+js技术分别演示了6中数学排序算法的基本原理，便想用ruby分别实现这六种算法。

[用HTML5实现的各种排序算法的动画比较](http://www.webhek.com/misc/comparison-sort/)

###冒泡排序

算法原理：

1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2. 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。在这一点，最后的元素应该会是最大的数。
3. 针对所有的元素重复以上的步骤，除了最后一个。
4. 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

{% highlight ruby linenos %}
#冒泡排序的算法复杂度为O(n^2)
def bubble_sort(list)
    list.each_index do |index|
        (list.length - index - 1).times do |e|
            if list[e] > list[e + 1]
                list[e], list[e + 1] = list[e + 1], list[e]
            end
        end
    end
end
{% endhighlight %}

###快速排序

算法原理：
快速排序是C.R.A.Hoare于1962年提出的一种划分交换排序。它采用了一种分治的策略，通常称其为分治法(Divide-and-ConquerMethod)。
步骤：

1. 从数列中挑出一个元素，称为 "基准"（pivot），
2. 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作。
3. 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

{% highlight ruby linenos %}
#快速排序的平均时间复杂度为O(nlogn)
def quicksort(list)
    return list if list.length <= 1
    pivot = list.pop
    less    ||= []
    greater ||= []
    (list - [pivot]).each do |element|
        element < pivot ? less << element : greater << element
    end
    quicksort(less) + [pivot] + quicksort(greater)
end
{% endhighlight %}

以上方法可以做进一步的简化：

{% highlight ruby linenos %}

def quicksort(list)
    return list if list.length <= 1
    pivot = list.pop
    #partition是一个迭代器，它返回两个数组，第一个数组包含了代码块中返回true的元素，第二个数组包含剩下的
    less, greater = list.partition { |e| e < pivot }
    quicksort(less) + [pivot] + quicksort(greater)
end
{% endhighlight %}

###二分查找

####算法要求

1. 必须采用顺序存储结构；
2. 数组必须是有序的；

####算法原理

折半查找法也称为二分查找法，它充分利用了元素间的次序关系，采用分治策略，可在最坏的情况下用O(log n)完成搜索任务。它的基本思想是，将n个元素分成个数大致相同的两半，取a[n/2]与欲查找的x作比较，如果x=a[n/2]则找到x，算法终止。如 果x<a[n/2]，则我们只要在数组a的左半部继续搜索x（这里假设数组元素呈升序排列）。如果x>a[n/2]，则我们只要在数组a的右 半部继续搜索x。

def binary_search(list, len, key)
    left, right = 0, len - 1
    while left <= right
        mid = (left + right) >> 1
        if list[mid] == key
            return mid
        elsif list[mid] > key
            right = mid -1
        else
            left = mid + 1
        end
    end
    return -1
end