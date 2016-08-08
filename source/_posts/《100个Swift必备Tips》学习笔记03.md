---
title: 《100个Swift必备Tips》学习笔记03
tags:
  - 笔记
  - Swift
category:
  - iOS开发
date: 2016-07-29 09:15:37
---

## 方法嵌套
可以在函数内部定义一个或者多个函数，当我们在一个函数中需要调用一个或者多个小函数时，我们可以直接把小函数定义在大函数的内部。

## 命名空间
OC 没用命名空间，所有的代码和引用的静态库最终都会被编译到同一个域和二进制中。当我们有重复的类名，会在编译的时候发送冲突，为解决这个问题一般会加上两到三个字母前缀，但还是有重复的几率。

在 Swift 可以使用命名空间了， 但是 Swift 的命名空间是基于 module (模块)的，也就是说同一个 target 里的类型还是不能重名。两种策略来解决：
**方式一**：可以通过创建 Cocoa (Touch) Framework 的 target 的方法来新建一个 module，这样我们就可以在两个不同的 target 中添加同样名字的类型了。

**方式二**：使用类型嵌套的方法来指定访问的范围。常见做法是将名字重复的类型定义到不同的 struct 中，以此避免冲突。这样在不适用多个 module 的情况下也能取得隔离同名类型的效果。

注：在一般开发中尽量不要定义名字相同的同类型变量。

<!--more-->

## typealias 
typealias 是用来为已经存在的类型重新定义名字的。目的是让代码变得更加清晰，更易阅读。

```swift
typealias Location = CGPoint
typealias Distance = Double

func distanceBetweenPoint(location: Location,
    toLocation: Location) -> Distance {
        let dx = Distance(location.x - toLocation.x)
        let dy = Distance(location.y - toLocation.y)
        return sqrt(dx * dx + dy * dy)
}

let origin: Location = Location(x: 0, y: 0)
let point: Location = Location(x: 1, y: 1)

let distance: Distance =  distanceBetweenPoint(origin, toLocation: point)”
```

##  可变参数函数
可变参数函数指的是可以接受任意多个参数的函数

```swift
// 声明
func sum(input: Int...) -> Int {
    //....
}
```
输入的 `input` 在函数体内部将被作为数组[Int]来使用，现在我们通过实现一个数组所有内容的累加来练习一下

```swift
/**
 * 四个函数是等价的，区别在与 reduce 函数的不同调用，由上至下分别 由全到简
 */
func sum1(input: Int...) -> Int {
    return input.reduce(0, combine: { (Total, num) -> Int in
        return Total + num
    })
}

func sum2(input: Int...) -> Int {
   return input.reduce(0) { Total, num -> Int in
        return Total + num
   }
}

func sum3(input: Int...) ->Int  {
    return input.reduce(0) { $0 + $1 }
}

func sum4(input: Int...) -> Int {
    return input.reduce(0, combine: +)
}
```

## Designated，Convenience 和 Required

* Swift 的初始化方法超级严格。强化了 `designated` 初始化方法的地位。Swift 中不加修饰的 init 方法都需要在方法中保证所有非 Optional 的实例变量被赋值初始化，而	在子类中也强制调用 super 版本的 `designated` 初始化，保证被初始化的对象总是可以被完整的初始化。

* `convenience` 初始化方法必须调用同一个类中的 `designated` 初始化完成设置。并且， convenience 的初始化方法是不能被子类重写或者是从子类中以 super 的方式被调用的。

* 对于默写我们希望子类中一定实现的 `designated` 初始化方法，我们可以通过添加 `required` 关键字进行限制，限制子类必须对这个方法重写实现。

## default 参数
Swift 的方法支持默认参数，即在声明方法时，可以给某个参数一个默认使用的值。在调用的时候如果传入了这个参数，则使用传入的值，如果缺少这个输入参数，那么直接使用设定的默认值进行调用。

```swift
func sayHello1(str1: String = "Hello", str2: String, str3: String) {
    print(str1 + str2 + str3)
}

func sayHello2(str1: String, str2: String, str3: String = "World") {
    print(str1 + str2 + str3)
}
```

调用

```swift
sayHello1(str2: " ", str3: "World")
sayHello2( str1: "Hello", str2: " ")”
```

## 正则表达式
Swift 目前没有在语言层面支持 `正则表达式`，所以我们想要自定义 "=~" 运算符来简化操作。

* 首先做一个封装


```swift
struct RegexHelper {
    let regex: NSRegularExpression
    
    init(pattern: String) throws {
        
        try regex = NSRegularExpression(pattern: pattern, options: .CaseInsensitive)
        
    }
    
    func match(input: String) -> Bool {
        let matches = regex.matchesInString(input, options: [], range: NSMakeRange(0, input.characters.count))
        
        return matches.count > 0
    }
}
```

* 然后再自定义操作符

```swift
infix operator =~ {
    associativity none
    precedence 130
}

func =~(lhs: String, rhs: String) -> Bool {
    do {
        return try RegexHelper(rhs).match(lhs)
    } catch _ {
        return false
    }
}
```

* 使用

```swift
if "onev@onevcat.com" =~ "^([a-z0-9_\\.-]+)@([\\da-z\\.-]+)\\.([a-z\\.]{2,6})$" {
    print("有效的邮箱地址")
}
```


