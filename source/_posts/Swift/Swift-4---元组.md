
---
title:  元组
categories:
- Swift
tags: 
- Swift
hide: true
---
#### 元组的比较是自左向右的,直至比较出结果大小,若全部相等,则元组相等
```
“Tuples are compared from left to right, one value at a time, until the comparison finds two values that aren’t equal. Those two values are compared, and the result of that comparison determines the overall result of the tuple comparison. If all the elements are equal, then the tuples themselves are equal. For example:”

摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)”。 iBooks. 
```
```
(1, "zebra") < (2, "apple")   // true because 1 is less than 2; "zebra" and "apple" are not compared
(3, "apple") < (3, "bird")    // true because 3 is equal to 3, and "apple" is less than "bird"
(4, "dog") == (4, "dog")      // true because 4 is equal to 4, and "dog" is equal to "dog"

摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)”。 iBooks. 
```

> “NOTE

> The Swift standard library includes tuple comparison operators for tuples with fewer than seven elements. To compare tuples with seven or more elements, you must implement the comparison operators yourself.”

> 摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)”。 iBooks. 
>

Swift 标准库对元组的比较仅支持小于7个元素的元组,如果一个元组内包含7个或更多元素的话,你必须自己实现比较操作.
