---
title: HashMap的实现原理
date: 2018-03-20 15:59:34
tags: [数据结构]
---


### HashMap实现原理

#### HashMap简介

HashMap 是一个散列表，允许使用空值和空键。存储的内容是键值对(key-value)映射

HashMap 继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口。
HashMap 的实现不是同步的，这意味着它不是线程安全的。它的key、value都可以为null。此外，HashMap中的映射不是有序的。

HashMap 的实例有两个参数影响其性能：“初始容量” 和 “加载因子”。容量 是哈希表中桶的数量，初始容量 只是哈希表在创建时的容量。加载因子 是哈希表在其容量自动增加之前可以达到多满的一种尺度。当哈希表中的条目数超出了加载因子与当前容量的乘积时，则要对该哈希表进行 rehash 操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数。
通常，默认加载因子是 0.75, 这是在时间和空间成本上寻求一种折衷。加载因子过高虽然减少了空间开销，但同时也增加了查询成本（在大多数 HashMap 类的操作中，包括 get 和 put 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少 rehash 操作次数。如果初始容量大于最大条目数除以加载因子，则不会发生 rehash 操作。

#### HashMap的数据结构

在 Java 编程语言中，最基本的结构就是两种，一个是数组，另外一个是指针（引用），HashMap 就是通过这两个数据结构进行实现。HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。


HashMap 底层就是一个数组结构，数组中的每一项又是一个链表。当新建一个 HashMap 的时候，就会初始化一个数组。


继承关系：

	java.lang.Object
	   ↳     java.util.AbstractMap<K, V>
	         ↳     java.util.HashMap<K, V>
	
	public class HashMap<K,V>
	    extends AbstractMap<K,V>
	    implements Map<K,V>, Cloneable, Serializable { }
    
#### HashMap的构造函数

	// 默认构造函数。
	HashMap()
	
	// 指定“容量大小”的构造函数
	HashMap(int capacity)
	
	// 指定“容量大小”和“加载因子”的构造函数
	HashMap(int capacity, float loadFactor)
	
	// 包含“子Map”的构造函数
	HashMap(Map<? extends K, ? extends V> map)