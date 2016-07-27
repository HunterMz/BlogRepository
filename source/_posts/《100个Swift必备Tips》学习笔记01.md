---
title: 《100个Swift必备Tips》学习笔记01
tags:
  - 笔记
  - Swift
category:
  - iOS开发
date: 2016-07-27 17:56:33
---

前言：趁着最近项目不是很忙，学习下喵神的《100个Swift必备Tips》，边学边做些笔记方便日后查阅，也希望能给其他和我一样学习 Swift 的开发者提供一点点帮助。

## Struct Mutable 的方法
如果想在 `struct` 内部通过函数修改变量的值，需要在函数前面加 `mutating`

## 将 protocol 的方法声明为 mutating
因为 `struct` 是 `Immutable` 的，所以为了使 `struct` 实现协议的方法，并在方法内修改 struct 的变量，需要将协议方法声明为 mutating

## Sequence 序列化
一个类如果实现了 `SequenceType` 协议，那么他就可以使用 `for … in` 语法进行遍历了，包括我们自己的定义的类。

那么如果还实现 `SequenceType` 协议呢？
<!--more-->
一步一步来，首先自定义一个实体类 Book ：     

``` swift
class Book {

    var name:String = ""
    var price:Float = 0.0

    init(name: String, price: Float) {

        self.name = name
        self.price = price
    }
}
```

接下来再定义一个 `ReverseGenerator` 类，它实现了 `GeneratorType` 协议.      

``` swift
class ReverseGenerator<T>: GeneratorType {
    typealias Element = T
    var list: [T]? = [T]()
    var currentIndex:Int = 0

    init(array: [T]) {
        self.list = array
    }

    func next() -> Element? {
        if currentIndex < list?.count {
            let element = list?[currentIndex]
            currentIndex += 1
            return element
        } else {
            return nil
        }
    }
}
```
最后我们来定义一个结构体，它实现了 `SequenceType`协议


``` swift
struct ReverseSequence<T>: SequenceType {
    var array: [T]
    
    init (array: [T]) {
        self.array = array
    }
    
    typealias Generator = ReverseGenerator<T>
    
    func generate() -> Generator {
        return ReverseGenerator(array: array)
    }
    
    mutating func add(element:T) {
        self.array .append(element)
    }
}
```

OK，现在让我们来爽一爽吧

``` swift
var bookList = ReverseSequence(array: [Book]())
    
bookList.add(Book(name: "《Swift从入门到放弃》", price: 250.0))
bookList.add(Book(name: "《iOS 开发宝典》", price: 998))
        
for book in bookList {
    print("\(book.name)----\(book.price)")
}
```

## 多元组的使用
交换两个变量的值

* 传统写法

``` swift
func swapMe<T>(inout a: T, inout b: T) {
    let temp = a
    a = b
    b = temp
}
```

* 文艺写法

``` swift
func swapMe<T>(inout a: T, inout b: T) {
    (a,b)= (b,a)
}
```

## @autoclosure 和 ??
### @autoclosure
`@autoclosure` 可以把一句表达式自动地封装成一个闭包 (closure)
需求：我们有一个方法要接受一个闭包，当闭包的执行结果为 true 时进行打印。

* 传统写法

```swift
func logIfTrue1(pridicate: () -> Bool) {
	if pridicate() {
		print("True")
	}
}

// 调用
logIfTrue1{ () -> Bool in
	return 3 > 2
}

// 简化后调用
logIfTrue1({3 > 2})

// 尾随闭包
logIfTrue1{3 > 2}
```

* 使用 `@autoclosure`

```swift
logIfTrue2(3>2)
```

### ??
`??` 可以判断左边的可选值是否为空，如果为空就返回右边的值

