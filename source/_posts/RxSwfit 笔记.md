---
title: RxSwift 操作符自查笔记
tags:
  - Swift
category:
  - iOS开发
date: 2016-09-02 14:39:33
---
# RxSwift 学习笔记
## Introduction

[项目Git链接](https://github.com/HunterMz/RxSwiftDemo)
在 Rx 的世界中，最根本的部分是 `观察者模式（Observer pattern）` 和 `迭代器模式（Iterator pattern）` 的结合。这两者是相互作用的，在两种情况下的计算都是从生产者中取出的值。对于迭代器模式，只要他们是可用的，我们就可以获取值;对于观察者模式，我们是在生产者给所有具体的观察者发出通知后，从中获取数据去处理值。

**观察者 + 迭代器 + 推送 = 可观测的（Observable）实体**

RxSwift核心概念就是一个观察者(Observer)订阅一个可观察序列(Observable)。观察者对 Observable 发射的数据或数据序列作出响应。可观察序列分为三种：
	* `.Next`: 序列的下一个元素
 	* `.Complete`: 表示事件序列的完结。
	* `.Error`: 同样表示完结，但是代表异常导致的完结。

**序列的冷热问题：**
	* 冷：只有当有观察者订阅这个序列时，序列才发射值。可以保证即使有订阅者中途加入也可以收到完整的事件序列。
	* 热：序列创建时就开始发射值。这样如果后面有订阅者来的时候，就可能会错过一些事件。

<!--more-->

### 创建一个序列有哪些方式：

* asObservable 返回一个序列
* create 使用 Swift 闭包的方式创建序列
* deferred 只有在有观察者订阅时，才去创建序列
* empty 创建一个空的序列，只发射一个 .Completed
* error 创建一个发射 error 终止的序列
* toObservable 使用 SequenceType 创建序列
* interval 创建一个每隔一段时间就发射的递增序列
* never 不创建序列，也不发送通知
* just 只创建包含一个元素的序列。换言之，只发送一个值和 .Completed
* of 通过一组元素创建一个序列
* range 创建一个有范围的递增序列
* repeatElement 创建一个发射重复值的序列
* timer 创建一个带延迟的序列

### empty
`empty` 是一个空的序列，它只发送 `.Completed` 消息。
	
``` swift
example("empty") {
    let emptySequence: Observable<Int> = empty()
    let subscription = emptySequence
        .subscribe { event in
            print(event)
        }
}
```

### never
`never` 是没有任何元素、也不会发送任何事件的空序列。

```swift
example("never") {
    let neverSequence: Observable<String> = never()
    let subscription = neverSequence
        .subscribe { _ in
            print("This block is never called.")
        }
}
```

### just
`just` 是只包含一个元素的序列，它会先发送 `.Next(value)` ，然后发送 `.Completed` 。

```swift
example("just") {
    let singleElementSequence = just(32)
    let subscription = singleElementSequence
        .subscribe { event in
            print(event)
        }
}

--- just example ---
Next(32)
Completed
```

### of
`of` 可以把一系列元素转换成事件序列。


```swift
example("of") {
    let sequenceOfElements/* : Observable<Int> */ = Observable.of(0, 1, 2, 3)
    let subscription = sequenceOfElements
        .subscribe { event in
            print(event)
        }
}

--- of example ---
Next(0)
Next(1)
Next(2)
Next(3)
Completed
```

### from
from 是通过 asObservable() 方法把 Swift 中的序列 (SequenceType) 转换成事件序列。

```swift
example("from") {
    let sequenceFromArray = [1, 2, 3, 4, 5].asObservable()
    let subscription = sequenceFromArray
        .subscribe { event in
            print(event)
        }
}

--- from example ---
Next(1)
Next(2)
Next(3)
Next(4)
Next(5)
Completed
```

### create
`create` 可以通过闭包创建序列，通过 `.on(e: Event)` 添加事件。

```swift
example("create") {
    let myJust = { (singleElement: Int) -> Observable<Int> in
        return create { observer in
            observer.on(.Next(singleElement))
            observer.on(.Completed)
            return NopDisposable.instance
        }
    }
    let subscription = myJust(5)
        .subscribe { event in
            print(event)
        }
}

--- create example ---
Next(5)
Completed
```

### failWith
`failWith` 创建一个没有元素的序列，只会发送失败 `(.Error)` 事件。

```swift
example("failWith") {
    let error = NSError(domain: "Test", code: -1, userInfo: nil)
    let erroredSequence: Observable<Int> = failWith(error)
    let subscription = erroredSequence
        .subscribe { event in
            print(event)
        }
}
--- failWith example ---
Error(Error Domain=Test Code=-1 "The operation couldn’t be completed. (Test error -1.)")
```

### deferred
`deferred` 会等到有订阅者的时候再通过工厂方法创建 `Observable` 对象，每个订阅者订阅的对象都是内容相同而完全独立的序列。

```swift
example("deferred") {
    let deferredSequence: Observable<Int> = deferred {
        print("creating")
        return create { observer in
            print("emmiting")
            observer.on(.Next(0))
            observer.on(.Next(1))
            observer.on(.Next(2))
            return NopDisposable.instance
        }
    }
    print("go")
    deferredSequence
        .subscribe { event in
            print(event)
        }
    deferredSequence
        .subscribe { event in
            print(event)
        }
}

--- deferred example ---
go
creating
emmiting
Next(0)
Next(1)
Next(2)
creating
emmiting
Next(0)
Next(1)
Next(2)
```

`defferd` 的其他用处：相当于一种延时加载，例如

```swift
example("defferd") { (_) in
    let dispose = DisposeBag()
    var value: String? = nil
    let subscription: Observable<String?> = Observable.deferred {
        return Observable.just(value)
    }
    // got value
    value = "Hello!"
    subscription.subscribe { event in
        print(event)
    }.addDisposableTo(dispose)
}
--- TestDeferred example ---
Next(nil)
Completed
``` 

如果使用 `deffered` 则可以正常显示想要的数据：

```swift
example("TestDeferred") {
    var value: String? = nil
    var subscription: Observable<String?> = deferred {
        return just(value)
    }
    // got value
    value = "Hello!"
    subscription.subscribe { event in
        print(event)
    }
}
--- TestDeferred example ---
Next(Optional("Hello!"))
Completed
```

## Subjects
`Subjects` 可以看做是一种代理和桥梁。它既是订阅者又是订阅源，这意味着它既可以订阅其他 `Observable` 对象，同事又可以对它的订阅者们发送事件。

定义一个辅助函数用于输出数据：

```swift
func writeSequenceToConsole<O: ObservableType>(name: String, sequence: O) {
    sequence
        .subscribe { e in
            print("Subscription: \(name), event: \(e)")
        }
}
```

### PublishSubject

`PublishSubject` 会发送订阅者从订阅之后的事件序列。

```swift
example("PublishSubject") {
    let subject = PublishSubject<String>()
    writeSequenceToConsole("1", sequence: subject)
    subject.on(.Next("a"))
    subject.on(.Next("b"))
    writeSequenceToConsole("2", sequence: subject)
    subject.on(.Next("c"))
    subject.on(.Next("d"))
}
--- PublishSubject example ---
Subscription: 1, event: Next(a)
Subscription: 1, event: Next(b)
Subscription: 1, event: Next(c)
Subscription: 2, event: Next(c)
Subscription: 1, event: Next(d)
Subscription: 2, event: Next(d)
```

### ReplaySubject

`ReplaySubject` 在新的订阅对象订阅的时候会补发所有已经发送过的数据队列， `bufferSize` 是缓冲区的大小，决定了补发队列的最大值。如果 `bufferSize` 是1，那么新的订阅者出现的时候就会补发上一个事件，如果是2，则补两个，以此类推。

```swift
example("ReplaySubject") {
    let subject = ReplaySubject<String>.create(bufferSize: 1)
    writeSequenceToConsole("1", sequence: subject)
    subject.on(.Next("a"))
    subject.on(.Next("b"))
    writeSequenceToConsole("2", sequence: subject)
    subject.on(.Next("c"))
    subject.on(.Next("d"))
}
--- ReplaySubject example ---
Subscription: 1, event: Next(a)
Subscription: 1, event: Next(b)
Subscription: 2, event: Next(b) // 补了一个 b
Subscription: 1, event: Next(c)
Subscription: 2, event: Next(c)
Subscription: 1, event: Next(d)
Subscription: 2, event: Next(d)
```

### BehaviorSubject 
`BehaviorSubject` 在新的订阅对象订阅的时候会发送最近发送的事件，如果没有则发送一个默认值。在新的订阅对象订阅的时候会发送最近发送的事件，如果没有则发送一个默认值。

```swift
example("BehaviorSubject") { 
    let subject = BehaviorSubject(value: "z") 
    writeSequenceToConsole("1", sequence: subject) 
    subject.on(.Next("a")) 
    subject.on(.Next("b")) 
    writeSequenceToConsole("2", sequence: subject) 
    subject.on(.Next("c")) 
    subject.on(.Completed) 
} 
 
--- BehaviorSubject example --- 
Subscription: 1, event: Next(z) 
Subscription: 1, event: Next(a) 
Subscription: 1, event: Next(b) 
Subscription: 2, event: Next(b) 
Subscription: 1, event: Next(c) 
Subscription: 2, event: Next(c) 
Subscription: 1, event: Completed 
Subscription: 2, event: Completed
```
### Variable

`Variable` 是基于 `BehaviorSubject` 的一层封装，它的优势是：不会被显式终结。即：不会收到 `.Completed` 和 `.Error` 这类的终结事件，它会主动在析构的时候发送 `.Complete` 。

```swift
example("Variable") {
    let variable = Variable("z")
    writeSequenceToConsole("1", sequence: variable)
    variable.value = "a"
    variable.value = "b"
    writeSequenceToConsole("2", sequence: variable)
    variable.value = "c"
}
--- Variable example ---
Subscription: 1, event: Next(z)
Subscription: 1, event: Next(a)
Subscription: 1, event: Next(b)
Subscription: 2, event: Next(b)
Subscription: 1, event: Next(c)
Subscription: 2, event: Next(c)
Subscription: 1, event: Completed
Subscription: 2, event: Completed
```

### map

`map` 就是对每个元素都用函数做一次转换，挨个映射一遍。

```swift
example("map") {
    let originalSequence = Observable.of(1,2,3)
    originalSequence
        .map { $0 * 2 }
        .subscribe { print($0) }
}
--- map example ---
Next(2)
Next(4)
Next(6)
Completed
```

### flatMap
`map` 在做转换的时候很容易出现『升维』的情况，即：转变之后，从一个序列变成了一个序列的序列。
什么是『升维』？在集合中我们可以举这样一个例子，我有一个好友列表 `[p1, p2, p3]`，那么如果要获取我好友的好友的列表，可以这样做：
`myFriends.map { $0.getFriends() }` 结果就成了 `[[p1-1, p1-2, p1-3], [p2-1], [p3-1, p3-2]]` ，这就成了好友的好友列表的列表了。这就是一个『升维』的例子。
在 Swift 中，我们可以用 `flatMap` 过滤掉 `map` 之后的 nil 结果。
在 Rx 中， `flatMap` 可以把一个序列转换成一组序列，然后再把这一组序列『拍扁』成一个序列。

```swift
example("flatMap") {
    let sequenceInt = Observable.of(1, 2, 3)
    let sequenceString = Observable.of("A", "B", "--")
            
    sequenceInt
        .flatMap { int in
            sequenceString
        }
        .subscribe {
            print($0)
        }
}

--- flatMap example ---
Next(A)
Next(B)
Next(--)
Next(A)
Next(B)
Next(--)
Next(A)
Next(B)
Next(--)
Completed
```

### scan

`scan`有点像 `reduce` ，它会把每次的运算结果累积起来，作为下一次运算的输入值。

```swift
example("scan") {
    let sequenceToSum = Observable.of(0, 1, 2, 3, 4, 5)
    sequenceToSum
        .scan(0) { acum, elem in
            acum + elem
        }
        .subscribe {
            print($0)
        }
}

--- scan example ---
Next(0)
Next(1)
Next(3)
Next(6)
Next(10)
Next(15)
Completed
```

### filter

filter 只会让符合条件的元素通过。

```swift
example("filter") {
    let subscription = Observable.of(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
        .filter {
            $0 % 2 == 0
        }
        .subscribe {
            print($0)
        }
}

--- filter example ---
Next(0)
Next(2)
Next(4)
Next(6)
Next(8)
Completed
```

### Combining

### zip

zip 人如其名，就是合并两条队列用的，不过它会等到两个队列的元素一一对应地凑齐了之后再合并，正如百折不撓的米斯特菜所提醒的， zip 就像是拉链一样，两根拉链拉着拉着合并到了一根上：

```swift
example("zip 1") {
    let intOb1 = PublishSubject<String>()
    let intOb2 = PublishSubject<Int>()
    zip(intOb1, intOb2) {
        "\($0) \($1)"
        }
        .subscribe {
            print($0)
        }
    intOb1.on(.Next("A"))
    intOb2.on(.Next(1))
    intOb1.on(.Next("B"))
    intOb1.on(.Next("C"))
    intOb2.on(.Next(2))
}
--- zip 1 example ---
Next(A 1)
Next(B 2)
```

### merge

`merge` 即 合并，把两个队列按照顺序组合在一起。

```swift
example("merge 1") {
    let subject1 = PublishSubject<Int>()
    let subject2 = PublishSubject<Int>()
    sequenceOf(subject1, subject2)
        .merge()
        .subscribeNext { int in
            print(int)
        }
    subject1.on(.Next(1))
    subject1.on(.Next(2))
    subject2.on(.Next(3))
    subject1.on(.Next(4))
    subject2.on(.Next(5))
}
--- merge 1 example ---
1
2
3
4
5
```

### combineLatest
如果存在两条事件队列，需要同时监听，那么每当有新的事件发生的时候，`combineLatest` 会将每个队列的最新的一个元素进行合并。

```swift
example("combineLatest 1") { 
    let intOb1 = PublishSubject<String>() 
    let intOb2 = PublishSubject<Int>() 
 
    combineLatest(intOb1, intOb2) { 
        "\($0) \($1)" 
        } 
        .subscribe { 
            print($0) 
        } 
 
    intOb1.on(.Next("A")) 
    intOb2.on(.Next(1)) 
    intOb1.on(.Next("B")) 
    intOb2.on(.Next(2)) 
} 
 
--- combineLatest 1 example --- 
Next(A 1) 
Next(B 1) 
Next(B 2)
```

### startWith
`startWith` 会在队列开始之前插入一个事件元素

```swift
example("startWith") { 
    let subscription = Observable.of(4, 5, 6) 
        .startWith(3) 
        .subscribe { 
            print($0) 
        } 
} 
 
--- startWith example --- 
Next(3) 
Next(4) 
Next(5) 
Next(6) 
Completed
```

### switch
当你的事件序列是一个事件序列的序列 (Observable<Observable>) 的时候，（可以理解成二维序列？），可以使用 switch 将序列的序列平铺成一维，并且在出现新的序列的时候，自动切换到最新的那个序列上。 和 merge 相似的是，它也是起到了将多个序列『拍平』成一条序列的作用。

```swift
example("switchLatest") { 
    let var1 = Variable(0) 
 
    let var2 = Variable(200) 
 
    // var3 is like an Observable<Observable<Int>> 
    let var3 = Variable(var1) 
 
    let d = var3 
        .switchLatest() 
        .subscribe { 
            print($0) 
        } 
 
    var1.value = 1 
    var1.value = 2 
    var1.value = 3 
    var1.value = 4 
 
    var3.value = var2 
    var2.value = 201 
    var1.value = 5 
 
    var3.value = var1 
    var2.value = 202 
    var1.value = 6 
} 
 
--- switchLatest example --- 
Next(0) 
Next(1) 
Next(2) 
Next(3) 
Next(4) 
Next(200) 
Next(201) 
Next(5) 
Next(6)
```

## 逻辑处理
### takeUntil
`takeUntil` 其实就是 `take` ，它会在终于等到那个事件之后触发 `.Completed` 事件。

```swift
example("takeUntil") { 
    let originalSequence = PublishSubject<Int>() 
    let whenThisSendsNextWorldStops = PublishSubject<Int>() 
 
    originalSequence 
        .takeUntil(whenThisSendsNextWorldStops) 
        .subscribe { 
            print($0) 
        } 
 
    originalSequence.on(.Next(1)) 
    originalSequence.on(.Next(2)) 
 
    whenThisSendsNextWorldStops.on(.Next(1)) 
 
    originalSequence.on(.Next(3)) 
} 
 
--- takeUntil example --- 
Next(1) 
Next(2) 
Completed 
```

### takeWhile
`takeWhile` 则是可以通过状态语句判断是否继续 `take`

```swift
example("takeWhile") { 
    let sequence = PublishSubject<Int>() 
    sequence 
        .takeWhile { int in 
            int < 2 
        } 
        .subscribe { 
            print($0) 
        } 
    sequence.on(.Next(1)) 
    sequence.on(.Next(2)) 
    sequence.on(.Next(3)) 
} 
 
--- takeWhile example --- 
Next(1) 
Completed
```
## Utility
针对事件序列的一些方法

### subscribe

`subscribe` 在前面已经接触过了，有新的事件就会触发。

```swift
example("subscribe") {
    let sequenceOfInts = PublishSubject<Int>()
    sequenceOfInts
        .subscribe {
            print($0)
        }
    sequenceOfInts.on(.Next(1))
    sequenceOfInts.on(.Completed)
}

--- subscribe example ---
Next(1)
Completed
```

### subscribeNext

`subscribeNext` 也是订阅，但是只订阅 `.Next` 事件。

```swift
example("subscribeNext") {
    let sequenceOfInts = PublishSubject<Int>()
    sequenceOfInts
        .subscribeNext {
            print($0)
        }
    sequenceOfInts.on(.Next(1))
    sequenceOfInts.on(.Completed)
}

--- subscribeNext example ---
1
```

### doOn

`doOn` 可以监听事件，并且在事件发生之前调用。

```swift
example("doOn") {
    let sequenceOfInts = PublishSubject<Int>()
    sequenceOfInts
        .doOn {
            print("Intercepted event \($0)")
        }
        .subscribe {
            print($0)
        }
    sequenceOfInts.on(.Next(1))
    sequenceOfInts.on(.Completed)
}
--- doOn example ---
Intercepted event Next(1)
Next(1)
Intercepted event Completed
Completed
```

### reduce

这里的 `reduce` 和 `CollectionType` 中的 `reduce` 是一个意思，都是指通过对一系列数据的运算最后生成一个结果。

```swift
example("reduce") {
    Observable.of(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
        .reduce(0, +)
        .subscribe {
            print($0)
        }
}
--- reduce example ---
Next(45)
Completed
```

### KVO

未使用 RxSwift 的 KVO

```swift
-(void)observeValueForKeyPath:(NSString *)keyPath
                     ofObject:(id)object
                       change:(NSDictionary *)change
                      context:(void *)context
```

使用 RxSwift 的 KVO

```swift
view.rx_observe(CGRect.self, "frame")
    .subscribeNext { frame in
        print("Got new frame \(frame)")
    }
```

### Notification
使用 RxSwift

```swift
NSNotificationCenter.defaultCenter()
    .rx_notification(UITextViewTextDidBeginEditingNotification, object: myTextView)
    .map { /*do something with data*/ }
```

## 错误处理
### catchError
`catchError` 可以捕获异常事件，并且在后面无缝接上另一段事件序列，丝毫没有异常的痕迹。

```swift
example("catchError 1") { 
    let sequenceThatFails = PublishSubject<Int>() 
    let recoverySequence = Observable.of(100, 200) 
 
    sequenceThatFails 
        .catchError { error in 
            return recoverySequence 
        } 
        .subscribe { 
            print($0) 
        } 
 
    sequenceThatFails.on(.Next(1)) 
    sequenceThatFails.on(.Next(2)) 
    sequenceThatFails.on(.Error(NSError(domain: "Test", code: 0, userInfo: nil))) 
} 
 
--- catchError 1 example --- 
Next(1) 
Next(2) 
Next(100) 
Next(200) 
Completed
```

### retry
`retry` 顾名思义，就是在出现异常的时候会再去从头订阅事件序列，妄图通过『从头再来』解决异常。

```swift
example("retry") {
    var count = 1 // bad practice, only for example purposes
    let funnyLookingSequence: Observable<Int> = create { observer in
        let error = NSError(domain: "Test", code: 0, userInfo: nil)
        observer.on(.Next(0))
        observer.on(.Next(1))
        if count < 2 {
            observer.on(.Error(error))
            count++
        }
        observer.on(.Next(2))
        observer.on(.Completed)
        return NopDisposable.instance
    }
    funnyLookingSequence
        .retry()
        .subscribe {
            print($0)
        }
}
--- retry example ---
Next(0)
Next(1)
Next(0)
Next(1)
Next(2)
Completed
```

## 学习资源
国内：
http://t.swift.gg/d/2-rxswift
http://blog.csdn.net/Hello_Hwc/article/details/51859330          

         
=========         

国外：
http://rx-marin.com/

http://rxmarbles.com/

http://www.thedroidsonroids.com/blog/ios/rxswift-by-examples-1-the-basics/
http://www.thedroidsonroids.com/blog/ios/rxswift-by-examples-2-observable-and-the-bind/
http://www.thedroidsonroids.com/blog/ios/rxswift-examples-3-networking/
http://www.thedroidsonroids.com/blog/ios/rxswift-examples-4-multithreading/
https://github.com/DroidsOnRoids/RxSwiftExamples


GitHub
https://github.com/DianQK/LearnRxSwift
https://github.com/devxoul/RxTodo


 

