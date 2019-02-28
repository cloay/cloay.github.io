---
layout: post
title: "iOS内存详解之内存管理"
date: 2019-02-28 23:45
comments: true
categories: ios,memory,retain,strong,release,arc
---
在了解了iOS了对象创建模式之后，接下来我们看看iOS的内存管理。通过理解其内部的原理来高效的使用内存、优化程序中的代码问题。在对iOS内存管理原理深入了解之前，我们先复习一下变量的生命周期和作用域，以及变量在内存中的分布。然后来看iOS是怎么对内存管理的以及怎么影响变量的生成和销毁的。

### 一、变量的生命周期
变量来源于数学，是计算机语言中能储存计算结果或能表示值抽象概念。变量可以通过变量名访问。是一种使用方便的占位符，用于引用变量数据在计算机内存中的地址。在指令式语言中，变量通常是可变的；但在纯函数式语言（如Haskell）中，变量可能是不可变（immutable）的。在一些语言中，变量可能被明确为是能表示可变状态。
<!-- more -->
变量分为局部变量、全局变量。在面向对象编程中，对于类而言，类包含局部变量，成员变量和类变量等，变量随着类的生成而生成，消亡而消亡。对于局部变量往往存在于函数或代码段中，其生命周期即其在函数声明开始在函数执行结束时结束。其作用域即为函数体或代码段{}。对于全局变量，如类的成员变量，其生命周期和整个类一样，作用域也是整个类。看下面代码，实际分析一下各变量的生命周期：
``` objective-c
//
//  SecondViewController.m
//  MemoryDemo
//
//  Created by cloay on 2018/11/2.
//  Copyright © 2018 cloay. All rights reserved.
//

#import "SecondViewController.h"
#import "WMButton.h"
#import "WMModel.h"

@interface WMSecondViewController ()
@property (nonatomic, strong) UIButton *globalBtn;  //全局成员变量
@property (nonatomic, assign) int instanceCount;
@end

@implementation WMSecondViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.instanceCount = 0;
    
    //局部变量1
    UIButton *localBtn = [[WMButton alloc] initWithFrame:CGRectMake(50, 120, 80, 44)];
    localBtn.tag = 1000;
    [localBtn setBackgroundColor:[UIColor grayColor]];
    [localBtn setTitle:@"局部变量" forState:UIControlStateNormal];
    [localBtn addTarget:self action:@selector(localBtnOnClick) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:localBtn];
    
    //局部变量2
    WMModel *model = [[WMModel alloc] init];
    model.name = @"局部变量";
    NSLog(@"%@ is created...", [model class]);
    
    self.globalBtn = [[WMButton alloc] initWithFrame:CGRectMake(50, 180, 80, 44)];
    self.globalBtn.tag = 1001;
    [self.globalBtn setBackgroundColor:[UIColor grayColor]];
    [self.globalBtn setTitle:@"点击0" forState:UIControlStateNormal];
    [self.globalBtn addTarget:self action:@selector(globalBtnOnClick) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:self.globalBtn];
    NSLog(@"func viewDidLoad is end...");
}

int count, count1; //全局变量
WMModel *globalModel;
- (void)globalBtnOnClick {
    self.block();
    globalModel = [[WMModel alloc] init];
    globalModel.name = @"全局变量";
    
    count += 1;
    count1 = 2;
    self.instanceCount += 1;
    [self.globalBtn setTitle:[NSString stringWithFormat:@"点击%d", count] forState:UIControlStateNormal];
}

- (void)localBtnOnClick {
    count = 0;
    [self.globalBtn setTitle:[NSString stringWithFormat:@"点击%d", count] forState:UIControlStateNormal];
    NSLog(@"localBtnOnClick...");
}


- (void)dealloc {
    NSLog(@"%@ dealloc ...", [self class]);
}

@end

```

从上述代码中我们可以看出定义了全局变量globalBtn它也是类SecondViewControler的成员变量，它的作用域是整个类中都可以访问，生命周期随着类的销毁而结束，在方法viewDidLoad中定义了局部变量localBtn和model。先看局部变量model的代码定义：
``` objective-c
#import "WMModel.h"

@implementation WMModel

- (void)dealloc {
    NSLog(@"%@ is dealloc ...", [self class]);
}
@end
```
只是简单重写了dealloc方法，根据之前我们的分析。model会随着方法viewDidLoad结束而被销毁。实际的结果也是这样的：

![model](/images/memory/3/model.png)

我们再看一下WMButton的代码：
``` objective-c
#import "WMButton.h"

@implementation WMButton

- (void)dealloc {
    NSLog(@"%ld WMButton is dealloc ...", self.tag);
}

@end
```
对于局部变量localBtn为什么没有打印被释放的log呢？不用着急，在第三部分我们详细说明。


### 二、变量在内存中的分布
在了解了变量的生命周期和作用域后，下面我们看看变量的在内存中是如何分布的。
根据之前第一篇的介绍程序在运行时会被操作系统从磁盘加载到内存中。程序启动后，程序中的变量在内存中会被划分到：栈区、堆区、全局区（静态区）、代码区、文字常量区这五个部分。这五个部分在内存的地址从高到低依次排列如下图：

![memory_region](/images/memory/3/memory_region.jpeg)
- 栈区：由编译器自动分配存放函数的参数值，局部变量的值等
- 堆区：由程序员分配释放，若程序员不释放，程序结束时 可能 有系统收回。
- 全局区（静态区：全局变量和静态变量是存储放在一块的，初始化的全局变量和静态变量在一个区域，未初始化的在相邻的另一个区域。程序结束后由系统释放。
- 代码区：存放代码的二进制数据的区域
- 文字常量区：即存放一些const申明的变量或者字符串常量等，该区域是只读的。

根据上述我们可以画出实例对象secondVc的内存结构如：

![secondvc_memory](/images/memory/3/secondVc_memory.png)

### 三、iOS内存管理

Objective-C使用引用计数管理内存。NSObject Protocol中定义了引用计数相关方法，在NSObject类中定义了dealloc方法，在对象释放时会自动调用。OC先前使用**MRR**（manual retain-release）即手动保留释放对象来管理内存，在iOS4.0时苹果推出了自动引用计数**ARC**（Automatic Reference Counting）来管理内存，编译器在编译时自动在代码中加入内存管理代码，不需要再手动控制。不管是MRR还是ARC其底层都是通过引用计数模型实现的。

**内存管理规则**

内存管理模型是基于对象的所有权。任何对象都可能拥有一个或多个所有者。只要一个对象至少有一个所有者，它就会继续在内存中存在。如果对象没有所有者，则运行时系统会自动销毁它。Cocoa设置以下策略即所有权规则： 

- 您拥有自己创建的任何对象 如使用 “alloc”, “new”, “copy”, or “mutableCopy”等方法生成的对象
- 您可以使用retain获取对象的所有权
通过该方法保证接收到的对象在接收到的方法中保持有效，并且该方法也可以安全地将对象返回给其调用者。您可以在两种情况下使用retain方法：

```
1. 在访问器方法或init方法的实现中，获取要作为属性值存储的对象的所有权;
2. 防止对象因某些其他操作的副作用而失效
```

- 当您不再需要它时，您必须放弃您拥有的对象的所有权 您通过向对象发送release消息或autorelease消息来放弃对象的所有权。因此，在Cocoa术语中，放弃对象的所有权通常被称为“释放”对象。通常在**dealloc**方法中，释放拥有的对象，如类的成员变量。
- 您必须放弃您不拥有的对象的所有权
这只是先前明确规定的政策规则的必然结果。

**对象所有权通过引用计数实现 - 通常在retain方法之后称为“保留计数” 。每个对象都有一个保留计数。**

- 创建对象时，其保留计数为1。
- 向对象发送retain消息时，其保留计数将增加1。
- 向对象发送release消息时，其保留计数减1。
- 向对象发送autorelease消息时，其保留计数在当前自动释放池块的末尾减1。
- 如果对象的保留计数减少到零，则将其取消分配。

内存管理通常都是对单个对象来说的，但实际上是管理是对象引用图:
<br>
![memory_management](/images/memory/3/memory_management_2x.png)
<br>

**将对象添加到集合（例如数组，字典或集合）时，集合将获得对象的所有权。当从集合中移除对象或集合本身被释放时，集合将放弃所有权。**
常见的场景都是显示的调用集合的相关方法，但还有一些情况如将view视图对象添加到view上时，调用父View的addSubview方法会隐式调用数组的addObject方法（个人猜测）。这时候父View会持有view子视图，直到父视图释放。在苹果的官方文档中对addSubview方法也有相关说明：

```
This method establishes a strong reference to view and sets its next responder to the receiver, which is its new superview.
```
所以localBtn直到secondVc释放了之后，才释放如图：
![localbtn](/images/memory/3/localbtn.png)






苹果强烈建议开发者使用ARC。因为使用ARC不需要手动添加release和retain方法，编译器会自动加入相关代码，在编译器内部使用保留和释放操作。使得开发人员把注意力集中到真正的代码编写中而无需关系内存管理问题。

虽然苹果建议我们使用ARC而不需要了解内存管理的底层原理，但这个建议大家还是不要听了！相信大家都深有体会！！！

<br>
<br>

**参考链接**

- [高级内存管理编程指南](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html#//apple_ref/doc/uid/10000011i)
- [关于内存地址和内存空间的理解](https://www.cnblogs.com/VIPler/p/4282584.html)


<br>
<br>
<br>