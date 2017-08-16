---
title: iOS基础知识总结一
tags:
  - 笔记
category:
  - iOS开发
date: 2017-08-16 20:00:33
---
# 堆栈
* 管理方式: 栈由编译器自动管理；堆由程序员手动管理。

* 大小： 
    * 栈：在Windows下，栈是向低地址扩展的数据结构，是一块连续的内存区域。意思是，栈顶的地址和栈的最大容量是系统预先规定好的。
    * 堆：堆是向高地址扩展的数据结构，是不连续的内存区域。这是由于系统是用链表来存储空闲内存地址的。而链表的遍历方向是由低地址向高地址。堆的大小取决于计算机的虚拟内存。由此可见，堆可以获得更大更灵活的内存空间。

* 碎片问题：
    对于堆来说，频繁的new/delete 会造成内存空间的不连续，从而造成大量的碎片，是程序效率降低。栈则不存在这个问题。因为栈是先进后出的队列，永远不可能有一个内存块从栈中间弹出。

* 分配方式：   
    堆是动态分配的，没有静态分配的堆。     
    栈有两种分配方式：
     * 静态分配：编译器完成的，比如局部变量的分配。
     * 动态分配：由 `alloc` 函数进行分配。区别于堆的动态分配的是，栈的动态分配由编译器完成，无需手动实现。
     
* 分配效率：
    栈是机器系统提供的数据结构，计算机底层对栈提供支持，分配专门的寄存器存放栈的地址，压栈出栈都有专门的指令执行，因此效率高；堆则是C/C++函数库提供的，机制很复杂。

# Core Foundation 的内存管理
 * 凡是带有Create、Copy、Retain 等字眼的函数，创建出来的对象，都需要在最后做一次release。
 * 比如：`CFRunLoopObserverCreate release`

# OC 的反射机制
* Class 反射：
    * 通过类名的字符串实例化对象：
    ```ObjectiveC
    Class tempClass = NSClassFromString(@"student");
    Student *stu = [[tempClass alloc] init]
    ```
    * 将类名变成字符串：
    ```
    Class class =[Student class]; 
    NSString *className = NSStringFromClass(class);
    ```
* SELL 的反射
    * 通过方法的字符串实例化方法：
    ```ObjectiveC
    SEL selector = NSSelectorFromClass(@"setName");  
    [stu performSelector:selector withObject:@"Mike"];
    ```
    * 将方法变成字符串
    ```ObjectiveC
    NSStringFomrSelector(@selector*(setName:))
    ```
    
# 协议中能否定义成员变量？
* 可以，但是只能在头文件中声明，编译器不会自动生成实例变量。需要自己处理 `getter`和`setter` 方法

# 如何理解Objective-C是动态运行时语言
* 将数据类型的确定、调用类别对象指定的方法由编译时，推迟到运行时。
* 多态：不同对象以自己的方式响应相同的消息。

# readwrite、readonly、 assign、 retain、 cop、nonatomic
* readwrite：可读可写.要生成getter 和 setter 方法；
* readonly：只读。成getter方法。属性不可在类外修改。
* assign：赋值。用于基本数据类型的属性修饰。是一个property的默认属性。
* copy：修饰string类型。在setter方法里，将传入的对象复制一份，再赋值。
* nonatomic：非原子操作，决定编译器生成的setter 和 getter 是否是原子操作。
* atomic：表示多线安全，一般不使用。
* retain：持有。setter方法中先保留参数，再赋值，并且传入参数的retaincount+1

* strong：修饰一般对象。和retain同义。先保留
* weak：编译器将为weak修饰的property生成带__weak所有权修饰符的实例变量。

# NotificationCenter、KVC、KVO、Delegate
* KVO：一对多，键值观察。
* KVC：键值编码，一个对象在调用setValue的时候:
```
graph TB
A[查找key对应的set方法]-->|YES| B[调用set方法]
A-->|NO| C[查找_key的成员变量是否存在]
C-->|YES| D[赋值]
C-->|NO| E[查找相同名称的key]
E--> |YES| F[赋值]
E--> |NO| G[调用`valueForUndefinedkey` 和 `setValue: forUndefinedKey`]
```
* Delegate：发送者和接受者是直接的一对一的关系。允许一个类在某些特定时刻通知到其他类，而不需要获取那些类的指针，接受者可以改变发送者的行为。
* Notification:观察者模式，通常发送者和接受者的关系是间接的多对多关系。消息的发送者告知接受者事件已经发生或者将要发送。接受者不能反过来影响发送者的行为。

* 区别     
    * delegate 效率比 NSNotification 高。
    * delegate 比 Notification 更直接，需要关注返回值。
    * 两个模块联系不是很紧密，就用NSNotification 传值，例如多线程间的传值。

# Category 和 Extention
* 类别：在没有原类 .m  文件之外给原类添加方法，但不能给原类添加成员变量，添加的成员变量不能初始化。
* 扩展：一种特殊形式的类别。在一个类的 .m 文件里声明变量和方法。可以给某个类添加私有方法和私有变量。
* 区别：
    * 扩展可以添加属性和方法，并且添加的方法是必须实现的。
    * 类别可以再不知道不改变原来代码的情况下往原有类添加新的方法，只能添加，不能删除修改。
    * 如果类别和原有类中的方法产生命名冲突，则类别的方法会覆盖原来的方法，即类别具有更高的优先级。

# 潜拷贝和深拷贝
* 潜拷贝：只复制对象的指针，而不复制对象本身，对象内容有改变，则复制之后的结果也会变。
* 深拷贝：复制对象本身。

# nil、Nil、NULL、NSNull  
* nil：针对Objective-C中的对象使用，表示空对象。nil对象调用任何方法表示什么也不执行，也不会崩溃。
* Nil：针对Objective-C中的类使用，标识空类（`Class myClass = Nil`）。
* NULL：对于C语言指针使用的，表示空指针。
* NSNull：类类型，用于标识空的占位对象，可以用来给服务端接口传空值。

# __block / weak 修饰的区别
* `__block` 在 arc 和 mrc 环境下都能用，可以修饰对象，也能修饰基本数据类型。
* `__weak` 只能用在 arc 环境下，只能修饰对象，不能修饰基本数据类型。
* `__block` 对象可以在Block 中重新赋值，`__weak` 不行。


