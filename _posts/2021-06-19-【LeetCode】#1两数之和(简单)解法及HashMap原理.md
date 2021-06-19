---
layout: post
title: 【LeetCode】#1两数之和(简单)解法及HashMap原理
date: 2021-06-19
tags: 算法与数据结构
---

本文主要用于记录学习算法和数据结构的笔记，内容主要来源于[阿飞爱学习-【LeetCode专题】#1 两数之和 —— 探究HashMap的原理及源码](https://segmentfault.com/a/1190000039004835)

## 题目

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 的那 两个 整数，并返回它们的数组下标。
你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。
你可以按任意顺序返回答案。

**示例 1：**  
**输入：**nums = [2,7,11,15], target = 9  
**输出：**[0,1]  
**解释：**因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。  

**示例 2：**  
**输入：**nums = [3,2,4], target = 6  
**输出：**[1,2]  

**示例 3：**  
**输入：**nums = [3,3], target = 6  
**输出：**[0,1]  

> **提示：**
>
> * 2 <= nums.length <= 10^4
> * -10^9 <= nums[i] <= 10^9
> * -10^9 <= target <= 10^9
> * 只会存在一个有效答案

来源：[力扣（LeetCode）](https://leetcode-cn.com/problems/two-sum)

## 解法

最容易想到的解法就是暴力枚举，两次for循环嵌套，空间复杂度O(1),但时间复杂度O(n^2)。  
更优的解法是基于哈希表(Java中HashMap),提高查询的效率，用空间来换时间，使时间和空间的复杂度均为O(n).

### 方法一：暴力枚举
```Java
public int[] twoSum(int[] nums, int target) {
  int[] result = new int[]{0, 1};
    if(nums.length==2){
        return result;
    }
    for (int i = 0; i < nums.length; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] == target) {
                result[0] = i;
                result[1] = j;
                return result;
            }
        }
    }
    // 没有返回值返回这个。
		throw new IllegalArgumentException("No two sum solution");
  }
```

### 方法二：两遍哈希表
```Java
/*
	 * 遍历两遍哈希表，第一次迭代将元素值和索引添加到表中， 第二次检查目标元素（target-nums[i]）是否存在表中。 目标元素不能是nums[i]本身。
	 */
	public int[] twoSums(int[] nums, int target) {
		Map<Integer, Integer> map = new HashMap<Integer, Integer>();
		// 迭代将元素值和索引添加到表中
		for (int i = 0; i < nums.length; i++) {
			// 数组里的项充当了key,for循环的i充当了value。
			map.put(nums[i], i);
		}
		// 迭代查找有没有target-nums[i]
		for (int i = 0; i < nums.length; i++) {
			// 判断这个值是否是map中的key
			int complement = target - nums[i];
			// 判断是否包含指定的键值，并且通过指定的键获取map的值（0，1，2）
			//看是否和这个for循环作用域里循环到的一样，防止同一个数的和等于结果。
			if (map.containsKey(complement) && map.get(complement) != i) {
				return new int[] { i, map.get(complement) };
			}
		}
		throw new IllegalArgumentException("No two sum solution");

	}
```

### 方法三：一遍哈希表
```Java
/*
	 * 一遍哈希表，在进行迭代并将元素插入到表中的同时，
	 * 还会回过头来检查表中是否已经存在当前元素所对应的目标元素
	 */
	public int[] twoSums(int[] nums, int target) {
		Map<Integer, Integer> map = new HashMap<Integer, Integer>();
		for (int i = 0; i < nums[i]; i++) {
			int complement = target - nums[i];
			// 因为最初map里并没有元素，所以直接跳过，put到map里，以后每次运行到这的时候，就会去已经有数据的map里查找。
			if (map.containsKey(complement)) {
				return new int[] { map.get(complement), i };
			}
			map.put(nums[i], i);
		}
		// 没有返回值返回这个。
		throw new IllegalArgumentException("No two sum solution");
	}
```

## 哈希表原理

### 1、为什么会有哈希表这种结构、它由什么组成？

我们已知的结构有数组和链表，两者各有优缺点：
**数组：** 数组存储结构是连续的，空间复杂度大，但查询的时间复杂度小。其寻址(通过下标搜索)效率高，一般的插入和删除效率低。即，**查询快，增删慢**。
**链表：** 链表存储结构是离散的，空间复杂度小。其寻址(通过下标搜索)效率低，一般的插入和删除效率高。即，**查询慢，增删快**。

**哈希表是基于数组和链表构建的，目的是为了充分发挥这两个结构的优势。**

对于一个哈希表，其包含四个部分的内容：

* key：即键值
* value：即数值
* hash：key对应的hash值，基于hash进行某种操作后得到的是表的index
* next：当index冲突时，存放冲突的数据，链表结构

### 2、为什么哈希表的查询效率会高于数组呢？

因为哈希表的结构也有点类似于字典。字典通过key可以快速查询到value，哈希表一样可以通过key快速查询到value，但其中间进行了散列变换（即hash计算），通过hash值来确定value所在的index，从而实现数据的获取。

> 一个确定的key可以计算得到唯一的hash值，而index是hash通过某种操作得到的，难以难免hash值的冲突，为了解决这个冲突才引入了链表，来高效的存储hash值相同的值。

### 3、hashMap.containsKey(value)的时间复杂度是多少？

HashMap在不同版本的JDK之间有着如下的区别：

* JDK 1.8以前 HashMap由数组和链表构成。
* JDK 1.8之后 HashMap由数组、链表和红黑树构成。

### 红黑树

红黑树的特点是查询快，增删慢。  
**红黑树** 作为一种二叉查找树，但它在二叉查找树的基础上增加了着色和相关的性质使得红黑树相对平衡，从而保证了红黑树的查找、插入、删除的时间复杂度最坏为 **O(log n)**。由于链表的查询时间复杂度为 **O(n)**，所以当链表很长时转换成红黑树将很好的提高效率！  

红黑树的引入是为了解决，当冲突发生时，链表太长而带来的查询性能下降的问题。在JDK 1.8中规定，当链表长度大于8后，链表转为红黑树结构。转换之前最坏的时间复杂度为 **O(n)**，转换之后的最坏时间复杂度为 **O(logn)**。  
HashMap的原理讲解推荐视频（p6-p9）：[https://www.bilibili.com/video/BV1WC4y1h7So?p=7](https://www.bilibili.com/video/BV1WC4y1h7So?p=7)
红黑树的原理推荐：[https://www.cnblogs.com/yyxt/p/4983967.html](https://www.cnblogs.com/yyxt/p/4983967.html)

**综上：**
JDK 1.8以前，hashMap.containsKey(value)最好情况便是O(1)，最坏情况是O(n)
JDK 1.8以后，hashMap.containsKey(value)最好情况便是O(1)，最坏情况是O(lgn)

### 4 HashMap源码分析
[参考文章](https://blog.csdn.net/qingtian_1993/article/details/80763381)
以及上面的[视频](https://www.bilibili.com/video/BV1WC4y1h7So?p=7)。
