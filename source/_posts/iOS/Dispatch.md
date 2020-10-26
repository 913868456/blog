---
title:  Dispatch 
date: 2017-10-27 22:05
categories:
- iOS
tags: 
- DispatchQueue 
---
# DispatchQueue
该类管理任务项的执行.每个提交到一个队列中的任务项将被系统管理的线程池处理.
[测试Demo](https://github.com/913868456/SwiftDemo)

## 同步和异步执行

每个任务项可以被同步或者异步执行.当一个任务项使用同步方法执行时,程序会直到执行项执行完毕才返回.当一个任务项使用异步方法执行时,异步方法会马上返回

## 串行和并行队列

一个分发队列可以是串行队列,任务项一次只能执行一个.或者是并行队列,任务项目被序列化,但是却可以一次全部运行或者在任何顺序下终止.
串行或者并行队列处理工作项都遵循FIFO原则

>  ## 重要
>  尝试同步执行工作项在主队列容易导致死锁.

## 全局队列(global concurrent queue)

全局队列是并行队列，系统创建了许多全局队列供程序使用


## DispatchQueue
Dispatchqueue  管理任务项的执行。每个提交到队列中的任务项被系统管理的线程池处理。

## 常用方法
~~~

实例创建

init(__label: UnsafePointer<Int8>?, attr: __OS_dispatch_queue_attr?)

参数介绍

label:  com.example.myqueue (可不填)，主要功能是标记线程，方便调试。
attr :  DISPATCH_QUEUE_SERIAL; DISPATCH_QUEUE_CONCURRENT; 标明创建的是串行队列还是并行队列。

init(label: String, qos: DispatchQoS, attributes: DispatchQueue.Attributes, autoreleaseFrequency: DispatchQueue.AutoreleaseFrequency, target: DispatchQueue?)

参数介绍

attributes: concurrent 可以用来创建并行队列

QoS  : 枚举（unspecified,background,utility,default,userInitiated,userInterractive）
优先级从低到高,在有优先级需求的时候使用.

target: 可以将队列添加到一个某个目标队列，最终将在目标队列执行。

实例方法

同步方法

// 提交block到一个队列中执行，直到那个block完成。
func sync(execute: () -> Void)

func sync(execute: DispatchWorkItem)

异步方法
func async(execute: DispatchWorkItem)

类属性
class var main: DispatchQueue

类方法
class func concurrentPerform(iterations: Int, execute work: (Int) -> Void)

iterations: 迭代次数

其他方法
func setTarget(queue: DispatchQueue?)
参数说明: 对象所在的新的目标队列.这个队列引用加一,之前所在队列,如果可能的话,将被释放.这个参数不能为NULL

 讨论
目标队列为对象处理负责.目标队列决定了对象的析构器的调用.更改对象的目标队列会改变他们的行为.
如果提交一个block到串行队列,串行队列的目标队列是一个不同的串行队列,该Block不会与提交到目标队列的其他Block并发调用或者说其他与该队列拥有相同目标队列的队列并发执行.

Dispatch souces: 分发源的目标队列明确了(事件处理和取消)的位置.

全局队列明确优先级,获取一个具有优先级别的全局队列.
class func global(qos: DispatchQoS.QoSClass)

~~~
用一个表格来表示串并行队列同步异步执行的情况

| | 同步执行| 异步执行
----|------|----
串行队列| 当前线程，一个一个执行| 其他线程，一个一个执行
并行队列| 当前线程，一个一个执行| 多个线程，一起执行 

# DispatchWorkItem

DispatchWorkItem包含能够被执行的任务。一个任务项能够被分配至DispatchQueue 和 DispatchGroup.一个任务项也能被置为一个DispatchSource 事件，注册项，或者错误处理。
```
实例化

init(qos: DispatchQoS = default, flags: DispatchWorkItemFlags = default, block: @escaping () -> Void)

参数介绍
Qos:   同上
flags: 任务项的性质，是否要创建新线程，或者创建barrior。
关于barrior需要描述一下,网上查找的资料描述: 只针对一个并行队列.同步点之前的任务，会并发执行，到了同步点就会等待，等待同步点的任务执行完成的时候，继续后面的任务，再次并发执行

项目中主要还是用到它的创建方法，其他略。
```

# DispatchTime

DispatchTime代表与具有纳秒（十亿分之一秒）精度的时钟相关的一个时间点。

~~~
实例方法
// 创建后即开始启动计时。
init(uptimeNanoseconds: UInt64)


实例属性
// 返回创建以来到当前时间的纳秒数，包含系统休眠的时间。
var uptimeNanoseconds: UInt64

类属性
static let distantFuture: DispatchTime

类方法
// 返回当前时间
static func now()
~~~
##  DispatchTimeInterVal
枚举类  使用其值来确定DispatchSourceTimer启动或者I/O处理的时间间隔。

（未完待续）
