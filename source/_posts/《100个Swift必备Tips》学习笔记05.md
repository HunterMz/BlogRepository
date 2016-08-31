---
title: 《100个Swift必备Tips》学习笔记05
tags:
  - 笔记
  - Swift
category:
  - iOS开发date: 2016-08-17 14:33:01
---

## 单例
在 Swift 单例的写法：

```swift
class MyManager  {
    private static let sharedInstance = MyManager()
    class var sharedManager : MyManager {
        return sharedInstance
    }
}
```

## 值类型和引用类型
Swift 的类型分为值类型和引用类型。值类型在传递和赋值的时进行复制，而引用类型则只会使用引用对象的一个"指向"。Swift 中的 `struct` 和 `enum`定义的类型是值类型，使用 `class` 定义的类型是引用类型。并且 Swift 中的所有内建类型都是值类型，比如 Int、Bool、String等。

## String 和 NSString




