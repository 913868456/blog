---
title:  Xcode Debug 问题
date: 2017-03-16 10:43
categories:
- iOS
tags: 
- Xcode 
---

在调试程序时，很多开发者喜欢用输出 log 的方式对代码的运行进行追踪，帮助理解。Swift 编译器并不会帮我们将 print 或者 debugPrint 删去，在最终 app 中它们会把内容输出到终端，造成性能的损失。我们当然可以在发布时用查找的方式将所有这些log 输出语句删除或者注释掉，但是更好的方法是通过添加条件编译来将这些语句排除在 Release 版本外。在 Xcode 的 Build Setting 中，在 Other Swift flags 的 Debug 栏中加入 -D DEBUG 即可加入一个编译标识。
![](https://onevcat.com/assets/images/2016/debug-flag.png)
之后我们就可以通过将` print` 或者 `debugPrint `包装一下：
```
     fuc Dlog (item : Any, file : String = #file, lineNum : Int = #line){
        #if DEBUG 
            let fileName = (file as NSString).lastPathComponent 
            print ("fileName:\(fileName)\n lineNum:\(lineNum)\n \(item)")
        #endif
     }
```

这样，在 Release 版本中，dPrint 将会是一个空方法，所有对这个方法的调用都会被编译器剔除掉。需要注意的是，在这种封装下，如果你传入的 items 是一个表达式而不是直接的变量的话，这个表达式还是会被先执行求值的。如果这对性能也产生了可测的影响的话，我们最好用 @autoclosure 修饰参数来重新包装 print。这可以将求值运行推迟到方法内部，这样在 Release 时这个求值也会被一并去掉：

```
      func Dlog(@autoclosure item: () -> Any, file : String = #file, lineNum : Int = #line) {
          #if DEBUG
               let fileName = (file as NSString).lastPathComponent 
               print ("fileName:\(fileName)\n lineNum:\(lineNum)\n \(item)")
          #endif
      }

      Dlog(resultFromHeavyWork())
      // Release 版本中 resultFromHeavyWork() 不会被执行
```
##参考
   [Swift 性能探索和优化分析](https://onevcat.com/2016/02/swift-performance/)
