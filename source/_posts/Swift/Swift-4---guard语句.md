
---
title:  guard语句
categories:
- Swift
tags: 
- Swift
hide: true

---
>“NOTE

>Constants and variables created with optional binding in an if statement are available only within the body of the if statement. In contrast, the constants and variables created with a guard statement are available in the lines of code that follow the guard statement, as described in Early Exit.”

>摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)”。 iBooks. 


if 语句中, 条件声明的变量和常量仅内部可用,而guard语句条件声明的变量和常量,在条件语句后面的代码中仍可用.
