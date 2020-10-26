---
title:  Error Handle
categories:
- Swift
tags: 
- Swift
hide: true
---

1.抛出错误的函数
```
   func canThrowAnError() throws {
    // this function may or may not throw an error
}
```


2.处理错误 do  catch 函数
```
do {
    try makeASandwich()
    eatASandwich()
} catch SandwichError.outOfCleanDishes {
    washDishes()
} catch SandwichError.missingIngredients(let ingredients) {
    buyGroceries(ingredients)
}
```
