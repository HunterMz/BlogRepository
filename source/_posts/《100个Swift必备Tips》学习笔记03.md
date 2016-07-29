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

注：个人觉得第一种实现方式有些麻烦，第二种方法还好，但在一般开发中尽量不要定义名字相同的同类型变量。

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

