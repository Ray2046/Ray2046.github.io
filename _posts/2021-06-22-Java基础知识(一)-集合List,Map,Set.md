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
