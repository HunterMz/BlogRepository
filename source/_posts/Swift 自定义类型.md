---
title: Swift——自定义类型
tags:
  - Swift
category:
  - iOS开发
date: 2016-07-21 14:29:33
---


## Hashable 协议
* 什么是Hashable(哈希值)？     
	`Hashable` 常用于快速定位和查找（用于Set里面的对象和Dictionary里面的作为key的对象，都需要实现（Hashable）协议）被哈希的对象。
	
	> 计算机内部会维护一个HashTable的一个表（想象成数据库的表），表类似于键值对的形式存在，hashValue就作为键（类似于带有索引的一个没有唯一约束的一个字段，注：并非主键，因为一个hashValue可能对应多条记录（也就是多个对象）。再判断相等用Equatable）
原文链接：http://www.jianshu.com/p/5aa75cd5e13e

* Swift 中的Hashable协议
	关于`Hashable` 协议，苹果官方文档中给出如下概述：
	
	> A hash value, provided by a type’s hashValue property, is an integer that is the same for any two instances that compare equally. That is, for two instances a and b of the same type, if a == b then a.hashValue == b.hashValue. The reverse is not true: Two instances with equal hash values are not necessarily equal to each other.
	由此可见通常可以通过比较两个类型的 `hash value` 来比较两个类型是否相等。
	
* Hashable 的应用
	因为 `Hashable` 本身继承自 `Equatable`，当自定义的类型遵守 `Hashable` 协议的时候还有实现 `Equatable` 的方法：
	
	``` swift
	func ==(lhs: GridPoint, rhs: GridPoint) -> Bool {
    	return lhs.x == rhs.x && lhs.y == rhs.y
	}

	extension GridPoint: Hashable {
    	var hashValue: Int {
        	return x.hashValue ^ y.hashValue
    	}
	}
	
	```
## Comparable协议
* 如果你的自定义类型想要实现比较大小，需要遵守 `Comparable`协议，因为 `Comparable`继承自 `Equatable`协议，所以你必须实现一个 `==`方法，又因为本身定义，至少要实现 `<` 方法，其他的操作符方法可选。例如：

	``` swift
	func >(lhs: BasicModel, rhs: BasicModel) -> Bool {
    	return lhs.age > rhs.age
	}

	func >=(lhs: BasicModel, rhs: BasicModel) -> Bool {
   	 return lhs.age >= rhs.age
	}

	func <(lhs: BasicModel, rhs: BasicModel) -> Bool {
    	return lhs.age < rhs.age
	}

	func <=(lhs: BasicModel, rhs: BasicModel) -> Bool {
    	return lhs.age <= rhs.age
	}

	```


