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

`ArrayList`：ArrayList是最常用的List实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。数组的缺点是每个元素之间不能有间隔，*所以占用的空间相对来说也会比较大*，当数组大小不满足时需要增加存储能力，就要讲已经有数组的数据复制到新的存储空间中。当从ArrayList的中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。
