---
title: 《100个Swift必备Tips》学习笔记04
tags:
  - 笔记
  - Swift
category:
  - iOS开发
date: 2016-08-08 17:38:17
---

# 内容概述
本篇主要包括   
* final 关键字的使用
* lazy 修饰符和 lazy 方法 
* 隐士解包 Optional
* Protocol Extension
* Where 的使用

<!--more-->

## final
final 关键字可以用在 class， func 或者 var 前面进行修饰，表示不允许对该内容进行继承或者重写操作。

* 使用场景
   * 类或者方法的功能确实已经完备了：比如一些工具类和方法，已经过完备验证，使用者只需要调用。
   * 子类继承和修改是一件危险的事情：如果一些方法在被继承或者重写后可能会导致一些破坏性的操作，则标记为 final 以确保稳定性。
   * 为了父类中某些代码一定会被执行：当父类中一些关键代码在被继承重写好偶必须执行（比如一些状态配置，认证等），可以使用一个 final 的方法来确保这些代码一定被执行。

```swift
class Parent {
    final func method() {
        print("开始配置")
        // ..必要的代码

        methodImpl()

        // ..必要的代码
        print("结束配置")
    }

    func methodImpl() {
        fatalError("子类必须实现这个方法")
        // 或者也可以给出默认实现
    }
}

class Child: Parent {
    override func methodImpl() {
    //..子类的业务逻辑
    }
}
``` 

## lazy 修饰符和 lazy 方法
* 修饰属性
在使用 lazy 作为属性修饰符时，只能声明属性是变量。另外我们需要显示地指定属性类型，并使用一个可以对这个属性进行赋值的语句来在首次访问属性时运行。这样，即使我们多次访问这个实例的 str 属性，也只有一次输出。

```swift
lazy var str: String = "Hello"
```

* lazy 方法
	可以使用 lazy 对 map 或是 filter 这类接收闭包并进行运行的方法一起，让整个行为变成延时进行的。先看下直接使用 map：
	
```swift
let data = 1...3
let result = data.map {
    (i: Int) -> Int in
        print("正在处理 \(i)")
        return i * 2
}
	
print("准备访问结果")
for i in result {
    print("操作后结果为 \(i)")
}
   
print("操作完毕")
```
	
输出为：
	
```swift
// 正在处理 1
// 正在处理 2
// 正在处理 3
// 准备访问结果
// 操作后结果为 2
// 操作后结果为 4
// 操作后结果为 6
// 操作完毕”
```

现在使用 lazy 进行修饰
	
```swift
let data = 1...3
let result = data.lazy.map {
    (i: Int) -> Int in
    print("正在处理 \(i)")
    return i * 2
}
	
print("准备访问结果")
for i in result {
    print("操作后结果为 \(i)")
}
   
print("操作完毕")
```
	
运行结果
	
```swift
// 准备访问结果
// 正在处理 1
// 操作后结果为 2
// 正在处理 2
// 操作后结果为 4
// 正在处理 3
// 操作后结果为 6
// 操作完毕
```
## 隐士解包 Optional

在声明的时候，在类型后一个感叹号 `!` 用来告诉编译器我们需要一个可以隐士解包的 Optional 值：

```swift
var maybeObeject: MyClass! = MyClass()
maybeObeject.fool()
   
// 这两个语句是等价的
maybeObeject!.fool()
      
// 如果是普通的 Optional 值，就不能这样写了
var maybeObeject: MyClass? = MyClass()
maybeObeject.fool() 
```

##  Protocol Extension
在 Swift 2中，我们可以对一个已有的 protocol 进行扩展，扩展中实现的方法将作为实现扩展的类型的默认实现。如下：

```swift
protocol Mypotocol {
    func method()
}

extension Mypotocol {
    func method() {
        print("Called")
    }
}
```

在实现这个借口的类型中，即使什么也不写，也可以编译通过。

```swift
struct MyStuct: Mypotocol {
    
}
```

## Where 的使用
* 在 switch 语句中使用

```swift
let name = ["王小二","张三","李四","王二小"]

name.forEach {
    switch $0 {
    case let x where x.hasPrefix("王"):
        print("\(x)是笔者本家")
    default:
        print("你好，\($0)")
    }
}

// 输出：
// 王小二是笔者本家
// 你好，张三
// 你好，李四
// 王二小是笔者本家”
```

* if let 或者 for 中使用

```swift
let num: [Int?] = [48, 99, nil]
num.forEach {
    if let score = $0 where score > 60 {
        print("及格啦 - \(score)")
    } else {
        print(":(")
    }
}

// 输出：
// :(
// 及格啦 - 99
// :(


for score in num where score > 60 {
    print("及格啦 - \(score)")
}

// 输出：
// 及格啦 - Optional(99)”
```

