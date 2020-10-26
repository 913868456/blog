
---
title:  函数
categories:
- Swift
tags: 
- Swift
hide: true
---
#### 函数参数标签和参数名

每个函数参数都有参数标签和参数名称。 调用函数时使用参数标签; 每个参数都写在函数调用中，其参数标签之前。 参数名称用于实现该函数。 默认情况下，**参数使用其参数名称作为参数标号.**

如果你不想给一个参数添加参数标签,你可以在该参数名前画一个(_)来替代参数标签;
```
func someFunction(_ firstParameterName: Int, secondParameterName: Int) {
    // In the function body, firstParameterName and secondParameterName
    // refer to the argument values for the first and second parameters.
}
someFunction(1, secondParameterName: 2)

摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)”。 iBooks. 
```

#### 默认参数值

在参数类型后面添加等号然后分配一个默认值,当调用该函数时,若不给其中有默认值得参数赋值,则参数值为默认值;

```
func someFunction(parameterWithoutDefault: Int, parameterWithDefault: Int = 12) {
    // If you omit the second argument when calling this function, then
    // the value of parameterWithDefault is 12 inside the function body.
}
someFunction(parameterWithoutDefault: 3, parameterWithDefault: 6) // parameterWithDefault is 6
someFunction(parameterWithoutDefault: 4) // parameterWithDefault is 12

摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)”。 iBooks. 
```
####  变量参数(Variadic Parameters)

```
func arithmeticMean(_ numbers: Double...) -> Double {
    var total: Double = 0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}
arithmeticMean(1, 2, 3, 4, 5)
// returns 3.0, which is the arithmetic mean of these five numbers

摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)”。 iBooks. 

```
> NOTE
> A function may have at most one variadic parameter.”
> 摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)。 iBooks.

#### In-Out Parameters 

函数参数默认是常量。 尝试从该函数体内更改函数参数的值会导致编译错误。 这意味着您不能错误地更改参数的值。 如果您想要一个函数修改参数的值，并且在函数调用结束后希望这些更改保持不变，请将该参数定义为in-out参数。

给in-out参数赋值时,必须为变量,并且在变量类型之前加(&),来证明这个参数是可以被函数改变的;

> NOTE
> In-out parameters cannot have default values, and variadic parameters cannot be marked as inout.”
> 摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)”。 iBooks. 

举例来讲,两个变量交换值

```
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}

var someInt = 3
var anotherInt = 107

swapTwoInts(&someInt, &anotherInt)

print("someInt is now \(someInt), and anotherInt is now \(anotherInt)")
// Prints "someInt is now 107, and anotherInt is now 3

摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)。 iBooks. 

```
