---
layout: post
title: "多线程实现之Operation Queues"
date: 2016-03-26 17:39
comments: true
categories: NSOperation,NSOperationQueue,多线程,Concurrency programming
---

###一、Operation Queues简介

Cocoa operations 为我们提供了一种面向对象的方式的来封装异步任务。包含Cocoa框架中的NSOperation和NSOperationQueue两个类，Operation通常与OperationQueue相结合来使用，是多线程的一种实现方式。其实现方式一是创建新线程，二是使用GCD的方式实现，相对于GCD可操作性高，是GCD的一种替代方式。
<!-- more -->
###二、NSOperation介绍
NSOperation是一个抽象类，表示一个独立的计算单元。因为是抽象类我们不能直接使用，苹果为我们实现了两个类既NSInvocationOperation和NSBlockOperation。我们也可以自定义自己的Operation通过继承NSOperation类。

####1、NSOperation的方法和属性
**NSOperation定义的属性如下：**

- completionBlock 该属性是任务完成后调用的block
- cancelled, executing, finished, concurrent, asynchronous, ready 这些是Operation的相关状态
- name Operation的名字
- queuePriority 执行的优先级
有以下几种：
- NSOperationQueuePriorityVeryLow
- NSOperationQueuePriorityLow
- NSOperationQueuePriorityNormal
- NSOperationQueuePriorityHigh
- NSOperationQueuePriorityVeryHigh

**NSOperation定义的方法有如下：**

- start	启动任务
- main		真正执行任务的的方法，自定义Operation必须实现该方法
- cancel	取消任务
- addDependency	添加依赖方法，添加依赖任务会在依赖任务完成后才执行
- removeDependency	移除依赖
- waitUnitlFinished 该方法会阻塞当前线程直到该操作任务完成。一个Operation决不能自身调用改方法，也不能在加入OperationQueue之前调用。

####2、创建Operation
使用NSInvocationOperation创建
```objective-c
NSInvocationOperation *invocationOperation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(invocationOperationTask) object:nil];
[invocationOperation start];
```
使用NSBlockOperation创建
```objective-c
NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"blockOperation thread =%@ ", [NSThread currentThread]);
        NSLog(@"blockOperation task...");
    }];
[blockOperation start];
```

自定义Operation必须实现initialization和main方法，Operation默认按synchronous方式执行，若要按asynchronous方式则必须实现start, isExecuting, isFinished, isConcurrent方法并且维护Executing, Finished, Concurrent等状态值的更新。

下面是自定义异步Operation实现：

```objective-c

#import "MyOperation.h"

@interface MyOperation(){
    BOOL finished;
    BOOL executing;
}

- (void)completeOperation;

@end

@implementation MyOperation

- (id)init {
    self = [super init];
    if (self) {
        executing = NO;
        finished = NO;
    }
    return self;
}

- (void)start{
    if ([self isCancelled]) {
        // Must move the operation to the finished state if it is canceled.
        [self willChangeValueForKey:@"isFinished"];
        finished = YES;
        [self didChangeValueForKey:@"isFinished"];
        NSLog(@"has cancelled ...");
        return;
    }
    
    // If the operation is not canceled, begin executing the task.
    [self willChangeValueForKey:@"isExecuting"];
    [NSThread detachNewThreadSelector:@selector(main) toTarget:self withObject:nil];
    executing = YES;
    [self didChangeValueForKey:@"isExecuting"];
    
}

- (void)main {
    @try {
        // Do the main work of the operation here.
        
        NSLog(@"do task ....");
        
        
        [self completeOperation];
    }
    @catch(...) {
        // Do not rethrow exceptions.
        NSLog(@"do task exception....");
    }
}

- (void)completeOperation {
    [self willChangeValueForKey:@"isFinished"];
    [self willChangeValueForKey:@"isExecuting"];
    
    executing = NO;
    finished = YES;
    
    [self didChangeValueForKey:@"isExecuting"];
    [self didChangeValueForKey:@"isFinished"];
}

- (BOOL)isConcurrent {
    return YES;
}

- (BOOL)isAsynchronous{
    return YES;
}

- (BOOL)isExecuting {
    return executing;
}

- (BOOL)isFinished {
    return finished;
}

@end

```

###三、使用NSOperationQueue管理Operation队列
简单的Operation不没有什么特别大的用处。在实际的项目中往往使用NSOperationQueue管理一连串的Operation来完成复杂任务。可以NSOperationQueue中的-(void)addOperation:(NSOperation *)方法将Operation加入队列中。

```objective-c
NSInvocationOperation *invocationOperation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(invocationOperationTask) object:nil];

NSBlockOperation *blockOperation = [NSBlockOperation blockOperationWithBlock:^{
        NSLog(@"blockOperation thread =%@ ", [NSThread currentThread]);
        NSLog(@"blockOperation task...");
    }];
    
MyOperation *myOperation = [[MyOperation alloc] init];
    
NSOperationQueue *myOperationQueue = [[NSOperationQueue alloc] init];
//将Operation加入到队列
[myOperationQueue addOperation:myOperation];
[myOperationQueue addOperation:invocationOperation];
[myOperationQueue addOperation:blockOperation];
    
```
也可以使用addOperationWithBlock方法快速添加一个Operation到OperationQueue中

```objective-c
[myOperationQueue addOperationWithBlock:^{
    NSLog(@"addOperationWithBlock thread =%@ ", 	[NSThread currentThread]);
    NSLog(@"addOperationWithBlock task...");
}];
```
加入到OperationQueue中的Operation会按自动执行。可以设置Operation的优先级既queuePriority来改变Operation执行的优先级。使用addDependency来添加依赖关系，变相控制Operation的执行顺序。扩展一下上面的代码：

```objective-c
NSOperationQueue *myOperationQueue = [[NSOperationQueue alloc] init];
[myOperationQueue setName:@"myqueue"];
    
NSBlockOperation *blockOperation1 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"blockOperation3 task start...");
    sleep(3);
    NSLog(@"blockOperation1 task finished...");
}];
    
NSBlockOperation *blockOperation2 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"blockOperation2 task start...");
    sleep(3);
    NSLog(@"blockOperation2 task finished...");
}];
    
NSBlockOperation *blockOperation3 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"blockOperation3 task start...");
    sleep(3);
    NSLog(@"blockOperation3 task finished...");
}];
[blockOperation3 addDependency:blockOperation1];
    
[myOperationQueue addOperation:blockOperation1];
[myOperationQueue addOperation:blockOperation2];
[myOperationQueue addOperation:blockOperation3];
    
NSLog(@"main thread ...");
```
我们为blockOperation3添加依赖blockOperation1，blockOperation3会在blockOperation1执行结束后才会启动。结果如下：

![operation-dependency](http://cloay.com/images/operation/operation-dependency.png)

在上文中我们介绍过waitUntilFinished方法下面我们就看一下它的用法，改变一下Operation加入队列的代码：

```objective-c
[myOperationQueue addOperation:blockOperation1];
[blockOperation1 waitUntilFinished];
[myOperationQueue addOperation:blockOperation2];
[myOperationQueue addOperation:blockOperation3];
```
我们插入了一行[blockOperation1 waitUntilFinished]代码，并将blockOperation3的依赖设置为blockOperation2。waitUntilFinished会阻塞当前线程直到blockOperation1执行完毕。

![operation-dependency](http://cloay.com/images/operation/operation-wait.png)

另外OperationQueue还有方法

```objective-c
- (void)addOperations:(NSArray<NSOperation *> *)ops
    waitUntilFinished:(BOOL)wait
```

通过改方法可以将一组Operation加入到队列中，wait为YES时表示需要等待任务执行结束会阻塞当前线程，为NO时则直接返回。修改代码如下：

```objective-c
NSOperationQueue *myOperationQueue = [[NSOperationQueue alloc] init];
[myOperationQueue setName:@"myqueue"];
    
NSBlockOperation *blockOperation1 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"blockOperation1 task start...");
    sleep(3);
    NSLog(@"blockOperation1 task finished...");
}];
    
NSBlockOperation *blockOperation2 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"blockOperation2 task start...");
    sleep(3);
    NSLog(@"blockOperation2 task finished...");
}];
    
NSBlockOperation *blockOperation3 = [NSBlockOperation blockOperationWithBlock:^{
    NSLog(@"blockOperation3 task start...");
    sleep(3);
    NSLog(@"blockOperation3 task finished...");
}];
[blockOperation3 addDependency:blockOperation2];
    
[myOperationQueue addOperations:[NSArray arrayWithObjects:blockOperation1, blockOperation2, blockOperation3, nil] waitUntilFinished:YES];
```
运行结果如下：

![operation-dependency](http://cloay.com/images/operation/operation-wait2.png)

我们可以取消一个Operation当Operation还没有执行的时候，只需要调用cancel方法或者调用OperationQueue中的cancelAllOperations取消队列中所有没有执行的Operation。

###四、总结 
通过NSOperation和NSOpertionQueue可以实现多线程的功能。我们可以通过继承NSOperation来自定义Operation来完成复杂的功能。NSOperationQueue和GCD都是苹果为我们提供的多线程实现方式，简化了我们处理复杂业务的难度。NSOperationQueue相对与GCD操作性更高，我们可以根据实际需要来选择。

####参考链接:
- [Concurrency Programming Guide](https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091-CH1-SW1)
- [NSOperation Class Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/NSOperation_class/index.html#//apple_ref/occ/instp/NSOperation/queuePriority)
- [NSOperationQueue Class Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/NSOperationQueue_class/index.html#//apple_ref/occ/cl/NSOperationQueue)

<br><br><br><br><br>