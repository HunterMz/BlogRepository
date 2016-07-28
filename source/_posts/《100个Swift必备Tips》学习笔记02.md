---
title: 《100个Swift必备Tips》学习笔记02
tags:
  - 笔记
  - Swift
category:
  - iOS开发
date: 2016-07-28 10:44:49
---
## Optional Chaining(可选链)
先上一段代码吧

```swift
class Toy {
    let name: String
    
    init(name: String) {
        self.name = name
    }
}

class Pet {
    var toy: Toy?
}

class Child {
    var pet: Pet?
}
```
<!--more-->
现在如果我们想访问 Toy 的 name 属性，一般来说会写这样的代码：

```swift
let xiaoming = Child()
xiaoming.pet?.toy?.name
```
在 `xiaoming.pet?.toy?.name` 可选链中任意一个 `?.` 的时候都有可能遇到 `nil` 而提前返回。

如果我们想让后面的操作更简洁一些，一般会用 Optional Binding (可选约束)来处理一下

```swift
if let toyName = xiaoming.pet?.toy?.name {
	print(toyName)
}
```
这样的操作是没有问题的，但是如果需求稍微变一下，就有点麻烦了，看下面

```swift
extension Toy {
    func play() {
    	print("我是玩具，快来玩我呀")
    }
}
```
如果有玩具的话就玩

```swift
xiaoming.pet?.toy?.play()
```
问题来了，如果除了小明，小红、小李、小王、老王等都想玩这个玩具，代码会变成下面的风格

```swift
xiaoming.pet?.toy?.play()
xiaohong.pet?.toy?.play()
xiaoli.pet?.toy?.play()
xiaowang.pet?.toy?.play()
laowang.pet?.toy?.play()
```
代码不是很美观，在这种时候我们会想要把这一串调用抽象出来，做一个闭包方便使用。传入一个 Child 对象，如果小朋友有宠物并且宠物有玩具的话，就去玩。

```swift

// 定义一个闭包
let playClosure = {(child: Child) -> Void? in
	child.pet?.toy?.play()
}

// 调用
// 使用 Optional Binding 来判断调用是否成功
if let result: Void = playClosure(xiaoming) {
	print("玩的好开心")
} else {
	print("没有玩具可以玩")
}
```

## 操作符
### 重载操作符
swift 允许我们重载操作符，在我们队一些特殊数据做运算时可以重载操作符来简化运算，例如对向量的操作：

```swift
struct Vector2D {
    var x = 0.0
    var y = 0.0
}

// 普通操作
let v1 = Vector2D(x: 2.0, y: 3.0)
let v2 = Vector2D(x: 1.0, y: 4.0)
let v3 = Vector2D(x: v1.x + v2.x, y: v1.y + v2.y)

// 重载操作符
func +(left: Vector2D, right: Vector2D) -> Vector2D {
    return Vector2D(x: left.x + right.x, y: left.y + right.y)
}

// 调用
let v4 = v1 + v2
```

### 自定义操作符
swift 中也允许我们定义一个全新的操作符，但是需要注意几个点

* inifx

>表示要定义的是一个中位操作符，即前后都是输入；其他的修饰还包括 prefix 和 postfix

* associativity

>定义了结合律，即如果多个同类的操作符顺序出现的计算顺序。比如常见的加法和减法都是 left，就是说多个加法同时出现时按照从左往右的顺序计算 (因为加法满足交换律，所以这个顺序无所谓，但是减法的话计算顺序就很重要了)。

* precedence

>运算的优先级，越高的话越优先进行运算。Swift 中乘法和除法的优先级是 150，加法和减法是 140，这里我们定义点积优先级 160，就是说应该早于普通的乘除进行运算

``` swift
func +* (left: Vector2D, right: Vector2D) -> Double {
    return left.x * right.x + left.y * right.y
}

infix operator +* {
    associativity none
    precedence 160
}
```
## func 的参数修饰
在 swift 中这种写法是错误的

```swift
// 错误代码
func incrementor(variable: Int) -> Int {
	return ++variable
}
```
> 因为在 swift 中所有有可能变化的地方，都默认认为不可变，也就是用 let 类声明。

需要做一下修改，才能编译通过且得到想要的结果

```swift
// 定义
func incrementor (inout varivable: Int) {
	++varivable
}

// 调用
var a = 3
incrementor(&a)
print(a)
```
参数修饰的传递

```swift
// 定义
func makeIncrementor(addNumber: Int) -> ((inout Int) -> Void) {
	func incrementor (inout variable: Int) {
		variable += addNumber
	}
   return incrementor
}

// 调用
let incrementThree = makeIncrementor(3)
incrementThree(&a)
print(a)
```
## 下标的使用
这章喵神主要演示了对 Array 扩展后是其变得更灵活。

```swift
extension Array {
    subscript(input: [Int]) -> ArraySlice<Element> {
        get {
            var result = ArraySlice<Element>()
            for i in input {
                assert(i < self.count, "Index out of range")
                result.append(self[i])
            }
            return result
        }

        set {
            for (index,i) in input.enumerate() {
                assert(i < self.count, "Index out of range")
                self[i] = newValue[index]
            }
        }
    }
}

// 使用
var arr = [1, 2, 3, 4, 5]

// 根据下标一次访问多个值
arr[[0,2,3]]      // [1, 3, 4]

// 根据下标一次修改多个值
arr[[0,2,3]]      // [-1, 2, -3, 4, 5]
```



