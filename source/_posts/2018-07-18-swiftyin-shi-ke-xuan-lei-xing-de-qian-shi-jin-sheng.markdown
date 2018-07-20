---
layout: post
title: "Swift隐式可选类型的前世今生"
date: 2018-07-18 10:40
comments: true
categories: Swift,iOS,Optional,Implicitly Unwrapped Optionals 
---
### Swift隐式可选类型的前世今生

#### 一、什么是隐式可选类型
在介绍隐式可选类型之前，我们先看看什么可选类型。在Swift中可选类型用来处理值可能不存在的情况，可选类型的意思是值可能有，也可能没有。我们看一个例子：
    
``` objective-c
let possibleNumber = "123"
let convertedNumber = Int(possibleNumber)
// convertedNumber 被推测为类型 "Int?"， 或者类型 "optional Int"
```
因为Int使用字符串的构造方法，可能成功也可能失败。比如字符串“Hello, world”就不能转换成Int。所以convertedNumber就是一个可选类型。当我们定义变量或常量时如果有值不存在的情况，使用可选类型。
<!-- more -->    
那么什么是隐式可选类型呢？隐式可选类型是可选类型的一种，有时候在程序架构中，第一次被赋值之后，可以确定一个可选类型总会有值。在这种情况下，每次都要判断和解析可选值是非常低效的，因为可以确定它总会有值。这种类型的可选状态被定义为隐式解析可选类型（implicitly unwrapped optionals）。使用！进行声明：
``` objective-c
let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString  // 不需要感叹号
``` 
隐式可选类型可以理解为一个可以自动解析的可选类型，因为它总是有值，不用使用！进行强制解包。
    
#### 二、可选类型底层实现     
我们先看看Swift源码中可选类型的定义：
``` objective-c
public enum Optional<Wrapped> : ExpressibleByNilLiteral {
    case none
    case some(Wrapped)
    /// Construct a non-\ `nil` instance that stores `some`.
    init(_ some: Wrapped)
}
```
可选类型被定义为枚举类型，因为Swift中枚举可以关联值，枚举类型可以明确的表示有值或者无值的情况。
我们再看看隐式可选类型的定义：
``` objective-c
public enum ImplicitlyUnwrappedOptional<Wrapped> : ExpressibleByNilLiteral {
    case none
    case some(Wrapped)
    /// Construct a non-\ `nil` instance that stores `some`.
    public init(_ some: Wrapped)

    /// Construct an instance from an explicitly unwrapped optional
    /// (`T?`).
    public init(nilLiteral: ())
}
```
虽然隐式可选类型可以当做一种有值的可选类型，但他们却被定义成了两种类型，没有继承关系。我们可以看到两者定义本身没有区别，因为隐式可选类型除了表示有值情况的可选类型，也可以当做正常的可选类型来使用。

#### 三、隐式可选类型的使用
什么情况下使用隐式可选类型呢？
1、在声明时无法初始化的常量定义为隐式可选类型，例如：
```objective-c
class MyView: UIView { 
    var startingOrigin: CGPoint! 
    override func awakeFromNib() { 
        super.awakeFromNib() 
        startingOrigin = frame.origin 
    } 
}
```
2、使用IB声明的属性
```objective-c
class MyView: UIView { 
    @IBOutlet private weak var titleLabel: UILabel! 

    override func layoutSubviews() { 
        super.layoutSubviews() 
        titleLabel.text = "Hello, world" 
    } 
}
```

总之，在使用隐式可选变量时保证它一定有值。
#### 四、隐式可选类型的今生
上面说的是隐式可选类型的前世，而在Swift4.2之后，Swift开发组重新实现可选类型的隐式解包。为什么要这么做，官方博客上说，“消除了类型检测中的一些矛盾，并且阐明了处理这些值的规则，使语义保持一致且易于推理。”具体可以参考[：动机](https://github.com/apple/swift-evolution/blob/master/proposals/0054-abolish-iuo.md#motivation)
重新实现之后，在使用隐式可选包时编译器不会再自动的解析，需要我们手动进行强制解析，即加上！。

在新的实现上，隐式可选类型和可选类型是同一种类型，类型检测的一致性将会更好，编译器的特殊情况也会更少。删除这些特殊情况会减少处理声明时的错误数量。总之，尽量不要使用隐式可选类型，改用 if let 和 guard let 来显式解包。如果确定有值，就用 ! 来显式强制解包。

#### 参考链接
- [The swift programming language](https://docs.swift.org/swift-book/LanguageGuide/TheBasics.html)
- [Uses for Implicitly Unwrapped Optionals in Swift](https://drewag.me/posts/2014/07/05/uses-for-implicitly-unwrapped-optionals-in-swift)
- [Reimplementation of Implicitly Unwrapped Optionals](https://swift.org/blog/iuo/)





