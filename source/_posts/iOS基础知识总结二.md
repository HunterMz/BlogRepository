---
title: iOS基础知识总结二
tags:
  - 笔记
category:
  - iOS开发
date: 2017-08-22 20:00:33
---
# NSString 用 `copy` 和 `strong` 修饰的区别
* OC 中 NSString 为不可变字符串的时候，用 `copy` 和 `strong` 都是只分配一次内存。但是如果使用 `copy` ,需要判断字符串是否是不可变字符串，如果是不可变字符串，就不再分配空间，如果是可变字符串才分派空间。如果程序中用到 `NSString` 的地方特别多，每一次都要先进行判断就会耗费性能。用 `strong` 就不会判断，所以在类型已知的情况下，可以直接使用 `strong`。

# NSArray、NSMutableArray、NSSet、NSMutableSet、NSDictionary、NSMutableDictionary
* 特点
    * NSArray：
    不可变数组，有序，只能存储对象类型，可通过索引直接访问元素，而且元素类型可以不一样，但是不能进行增、删、改操作；
    * NSMutableArray:
    可变数组，能进行增、删、改操作。通过索引查询值很快，但是插入、删除效率很低。
    * NSSet：
    不可变集合，确定、互异、无序。只能访问不能修改。
    * NSMutableSet:
    可变集合，可以对集合增、删、改操作。集合通过值查询很快，插入、删除操作极快。
    * NSDictionary:
    不可变字典，无序，每个key对应的值唯一，可以通过key取值；
    * NSMutableDictionary:
    可变字典，能对字典进行增、删、改操作。通过key查询、插入、删除值都很快。
* 应用场景
    * 数组用于处理一组有序的数据集，比如常用列表的 dataSource 要求有序，可通过索引直接访问，效率高。
    * 集合要求具有确定性、互异性、无序性，在开发中少用到。
    * 字典是键值对集合，操作字典效率高，时间复杂度为常量，但是值是无序的。
  
    <!--more-->
    
# instancetype 和 id
* 都可以作为方法的返回类型。
* `instancetype` 可以返回和方法所在类相同类型的对象，`id` 只能返回未知类型的对象。简单来说就是 `instancetype` 在编译期确定实例的类型，而 `id` 在运行时检查类型。
* `instancetype` 只能作为返回值，不能像 `id` 那样作为参数。

# 野指针、内存泄露、僵尸对象、空指针
* 野指针：指针变量没有进行初始化或者指向的空间已经被释放。
    * 使用野指针调用方法，会报异常，崩溃。
    * 通常在调用玩 `release` 方法后，把保存对象的地址清空，赋值为nil, OC 中没有空指针异常，所以 `[nil retain]` 调用方法是不会有异常。

* 内存泄露：
    * 如 `Person *person = [Person new]` （对象提前赋值nil, 或者清空），在栈区的 `person` 已经被释放了，而堆区new产生的对象还没有被释放，就会造成内存泄露。
    * 在MRC模式下，没有配对释放或者对象提前赋值nil导致release不起作用，会引起内存泄露。
    
* 僵尸对象：堆中已经被释放的对象。
* 空指针：指针赋值为空，nil。

# 自动释放池是什么？如何工作？
* 自动释放池是用来存储多个对象类型的指针变量。
* 存入自动释放池的对象在自动释放池被销毁时，会对池内对象全部做一次release操作。
* 调用对象的 `autorelease` 方法可以把对象加入池中。
* 自动释放池在当前的 `runloop` 迭代结束时释放。一旦监听到 `Runloop` 即将进入睡眠等待状态，就释放自动释放池。
* 多次调用对象的 `autorelease` 会导致 `野指针异常`。

# 如何检查内存泄露
* 静态分析：analyze
* instruments 工具里有个leak可以动态分析，如果在 Block 中多次使用 weakSelf 的话，可以在 Block 中先使用 strongSelf，防止 Block 执行时 weakSelf 被意外释放。

# 内存缓存机制实现
* FIFO：新数据插入FIFO队列的尾部；数据在FIFO队列中顺序移动，淘汰队列头部的数据。
* LRU：新数据插入链表头部；每当缓存数据命中，则将数据移到链表头部；当链表满的时候，将链表尾部的数据丢弃。
* LFU：新数据插入队列尾部；队列中的数据被访问后，引用计数增加，队列重新排序；当需要淘汰数据时，将已经排序的列表最后的数据块删除。

# 常见的内存引用场景
* 定时器：`NSTimer` 经常被作为某个类的成员变量，而 `NSTimer` 初始化时要指定 self 为 target ，容易造成循环引用 `self ——>timer-->self` 另外，若timer一直处于 validate 的状态，则其引用计数将始终大于 0， 因此在不再使用定时器以后，应该先调用 `invalidate` 方法。
* Block：Block 在 copy 时都会对Block内部用的对象进行强引用。`self-->block-->self` 或者 `self-->block--_ivar(成员变量)`。
* 代理：在声明 `delegate` 时，使用 weak.

# KVO 的底层实现
* KVO 基于 runtime 机制实现。

* 当一个对象(假设是 person 对象， person 是 MYPerson)的属性值发生改变时，系统会自动生成一个类(继承自 MYPerson) NSKVONotifying_MYPerson，在这个类的 `setAge` 方法里，调用`[self willChangeValueForKey:@"age"] 和 [self didChangeValueForKey:@"age"]`，而这两个方法内部会主动调用监听者内部的 `- (void)observeValueForKeyPath` 这个方法。

* 当添加监听者后，person类型由 MYPerson 被改变成 NSKVONotifying_MYPerson




