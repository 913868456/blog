
---
title:  闭包
categories:
- Swift
tags: 
- Swift
hide: true
---

#### 闭包的三种形式

- 全局函数是一种没有函数名但是不捕获任何值的闭包 
- 嵌套函数是一种有函数名且从上级包裹的函数中捕获值得闭包
- 闭包表达式是一种没有名字,但可以从周边上下文中捕获值,使用轻量级语法书写的闭包

闭包表达式的一般形式

```
{ (parameters) -> returnType in
     statements
}

```

#### 逃逸闭包
特点: 包含闭包的函数返回之后才执行.
功能: 常见于异步数据回调处理,例如网络请求回调.

当一个闭包作为参数传到一个函数中，但是这个闭包在函数返回之后才被执行，我们称该闭包从函数中逃逸。当 你定义接受闭包作为参数的函数时，你可以在参数名之前标注 @escaping ，用来指明这个闭包是允许“逃逸”出 这个函数的

举个例子，很多启动异 步操作的函数接受一个闭包参数作为 completion handler。这类函数会在异步操作开始之后立刻返回，但是闭包 直到异步操作结束后才会被调用。在这种情况下，闭包需要“逃逸”出函数，因为闭包需要在函数返回之后被调用。

#### 自动闭包

特点:   不接收参数,被调用的时候才返回表达式的值.
功能:   延迟求值,减少计算成本.

自动闭包是一种自动创建的闭包，用于包装传递给函数作为参数的表达式。这种闭包不接受任何参数，当它被调用的时候，会返回被包装在其中的表达式的值。这种便利语法让你能够省略闭包的花括号，用一个普通的表达式
来代替显式的闭包。

自动闭包让你能够延迟求值，因为直到你调用这个闭包，代码段才会被执行。延迟求值对于那些有副作用(Side Effect)和高计算成本的代码来说是很有益处的，因为它使得你能控制代码的执行时机。

```
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"] 
print(customersInLine.count)
// 打印出 "5"
let customerProvider = { customersInLine.remove(at: 0) } 
print(customersInLine.count)
// 打印出 "5"
print("Now serving \(customerProvider())!") // Prints "Now serving Chris!"
print(customersInLine.count)
// 打印出 "4"
```
