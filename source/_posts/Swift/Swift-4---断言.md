
---
title:  断言(Assert)
categories:
- Swift
tags: 
- Swift
hide: true
---
#### 断言的使用可以在某些条件不满足的情况下,终止应用的执行

Assert 函数
```
assert(_:_:file:line:)  
//判断条件的真和假
```
例子

```
let age = -3
assert(age >= 0, "A person's age can't be less than zero.")
// This assertion fails because -3 is not >= 0.”

```

AssertionFailure 函数
```
assertionFailure(_:file:line:)
//已知条件为假
```
例子
```
if age > 10 {
    print("You can ride the roller-coaster or the ferris wheel.")
} else if age > 0 {
    print("You can ride the ferris wheel.")
} else {
    assertionFailure("A person's age can't be less than zero.")
}

```
Precondition 函数

```
precondition(_:_:file:line:)

preconditionFailure(_:file:line:)
```

> Note 

> Assert 仅在开发版本有效,Preconditon在开发和生产版本中都有效.


错误处理还可以使用fatalError函数
```
fatalError(_:file:line:)
```
> NOTE

> “如果以未选中的模式（-Ounchecked）进行编译，则不会检查Precondition。 编译器假定Precondition始终为真，并且相应地优化您的代码。 但是，无论优化设置如何，fatalError（_：file：line :)函数都会暂停执行。
