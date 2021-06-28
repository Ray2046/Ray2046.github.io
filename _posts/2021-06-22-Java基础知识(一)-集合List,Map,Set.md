---
layout: post
title: Java基础知识(一)-集合List,Map,Set
date: 2021-06-22
tags: Java   
---

## 一、集合是什么
Java集合类存放在java.util包中，是一个用来存放**对象**的容器。  
注意：  
　　　　1.集合只能存放对象。比如你存入一个int型数据66放入集合中，其实它是自动转换成Integer类后存入的，Java中每一种基本数据类型都有对应的引用类型。  
　　　　2.集合存放的都是对象的引用，而非对象本身。所以我们称集合中的对象就是集合中对象的引用。对象本身还是放在堆内存中。  
　　　　3.集合可以存放不同类型，不限数量的数据类型。

## 二、Java集合框架
![cmd-markdown-logo](https://img-blog.csdnimg.cn/20190310181825411.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Rpd2Vpa2FuZw==,size_16,color_FFFFFF,t_70)

上述所有的集合类，除了map系列的集合，即左边的集合都实现了Iterator接口。
* Iterator是一个用来遍历集合中元素的接口，主要有hashNext()，next()，remove()三种方法。
* 它的子接口ListIterator在它的基础上又添加了三种方法，分别是add()，previous()，hasPrevious()。

### 集合三种类型  
* `List`：一种有序列表的集合，例如，按索引排列的Student的List；
* `Set`：一种保证没有重复元素的集合，例如，所有无重复名称的Student的Set；
* `Map`：一种通过键值（key-value）查找的映射表集合，例如，根据Student的name查找对应Student的Map。

由于Java的集合设计非常久远，中间经历过大规模改进，我们要注意到有一小部分集合类是遗留类，不应该继续使用：  
* `Hashtable`：一种线程安全的Map实现；
* `Vector`：一种线程安全的List实现；
* `Stack`：基于Vector实现的LIFO的栈。
还有一小部分接口是遗留接口，也不应该继续使用：  
* `Enumeration<E>`：已被Iterator<E>取代。


## 三、List
> List是有序的Collection。  
> List一共三个实现类：分别是ArrayList、Vector和LinkedList。

`ArrayList`：ArrayList是最常用的List实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。数组的缺点是每个元素之间不能有间隔，当数组大小不满足时需要增加存储能力，就要讲已经有数组的数据复制到新的存储空间中。当从ArrayList的中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。
`Vector`：Vector与ArrayList一样，也是通过数组实现的，不同的是它支持线程的同步，即某一时刻只有一个线程能够写Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此，访问它比访问ArrayList慢。
`LinkedList`：LinkedList是用链表结构存储数据的，很适合数据的动态插入和删除，随机访问和遍历速度比较慢。另外，他还提供了List接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作堆栈、队列和双向队列使用。

说明：  
* 1.ArrayList在内存不够时默认是扩展50% + 1个，Vector是默认扩展1倍。  
* 2.Vector属于线程安全级别的，但是大多数情况下不使用Vector，因为线程安全需要更大的系统开销。  
* 3.一般使用ArrayList和LinkedList比较多。  
* 4.对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。  
* 5.对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据

表格对比`ArrayList`和`LinkedList`：   

| 操作              | ArrayList   | LinkedList    |  
| ----          | ----     | ----       |  
| 获取指定元素       | 速度很快     | 需要从头开始查找元素 |  
| 添加元素到末尾     | 速度很快     | 速度很快      |  
| 在指定位置添加/删除 | 需要移动元素 | 不需要移动元素 |  
| 内存占用	        | 少          | 较大         |  

通常情况下，我们总是优先使用`ArrayList`。

要正确使用`List`的`contains()`、`indexOf()`这些方法, 放入的实例必须正确覆写equals()方法，否则，放进去的实例，查找不到。我们之所以能正常放入`String`、`Integer`这些对象，是因为Java标准库定义的这些类已经正确实现了`equals()`方法。

### 要简化引用类型的比较，我们使用 ** Objects.equals() ** 静态方法：

```Java
public boolean equals(Object o) {
    if (o instanceof Person) {
        Person p = (Person) o;
        return Objects.equals(this.name, p.name) && this.age == p.age;
    }
    return false;
}
```


## 四、Map
正确使用`Map`必须保证：  
1.作为`key`的对象必须正确覆写`equals()`方法，相等的两个`key`实例调用`equals()`必须返回`true`；
2.作为`key`的对象还必须正确覆写`hashCode()`方法，且`hashCode()`方法要严格遵循以下规范：  
* 如果两个对象相等，则两个对象的`hashCode()`必须相等；
* 如果两个对象不相等，则两个对象的`hashCode()`尽量不要相等。  
上述第一条规范是正确性，必须保证实现，否则`HashMap`不能正常工作。
而第二条如果尽量满足，则可以保证查询效率，因为不同的对象，如果返回相同的`hashCode()`，会造成Map内部存储冲突，使存取的效率下降。
和实现`equals()`方法遇到的问题类似，为了避免`NullPointerException`,我们在计算hashCode()的时候，经常借助Objects.hash()来计算：  
```Java
int hashCode() {
    return Objects.hash(firstName, lastName, age);
}
```
所以，编写`equals()`和`hashCode()`遵循的原则是：  
`equals()`用到的用于比较的每一个字段，都必须在`hashCode()`中用于计算；`equals()`中没有使用到的字段，绝不可放在`hashCode()`中计算。
另外注意，对于放入`HashMap`的`value`对象，没有任何要求。

### HashMap的第一个问题
既然`HashMap`内部使用了数组，通过计算`key`的`hashCode()`直接定位`value`所在的索引，那么第一个问题来了：`hashCode()`返回的`int`范围高达±21亿，先不考虑负数，`HashMap`内部使用的数组得有多大？  

实际上`HashMap`初始化时默认的数组大小只有16，任何`key`，无论它的`hashCode()`有多大，都可以简单地通过：  
```Java
int index = key.hashCode() & 0xf; // 0xf = 15 （TODO:这个地方要看看hencoder）
```
虽然指定容量是`10000`，但`HashMap`内部的数组长度总是`2^n`，因此，实际数组长度被初始化为比`10000`大的`16384`（2^14）。

### HashMap的第二个问题
最后一个问题，key的`hash冲突`。
在冲突的时候，一种最简单的解决办法是用`List`（`List<Entry<String, Person>>`）存储`hashCode()`相同的`key-value`。显然，如果冲突的概率越大，这个`List`就越长，`Map`的`get()`方法效率就越低，这就是为什么要尽量满足条件二。

### 使用EnumMap
`EnumMap`的`key`对象是`enum`类型。
它在内部以一个非常紧凑的数组存储`value`，并且根据`enum`类型的`key`直接定位到内部数组的索引，并不需要计算`hashCode()`，不但效率最高，而且没有额外的空间浪费。

### 使用TreeMap

`HashMap`是一种以空间换时间的映射表，它的实现原理决定了内部的Key是无序的.  
还有一种`Map`，它在内部会对Key进行排序，这种Map就是`SortedMap`。注意到`SortedMap`是**接口**，它的实现类是`TreeMap`。  
`SortedMap`保证遍历时以Key的顺序来进行排序。例如，放入的Key是`"apple"`、`"pear"`、`"orange"`，遍历的顺序一定是`"apple"`、`"orange"`、`"pear"`，因为`String`默认按字母排序.  
使用`TreeMap`时，放入的Key必须实现`Comparable`接口。`String`、`Integer`这些类已经实现了`Comparable`接口，因此可以直接作为Key使用。作为Value的对象则没有任何要求。
