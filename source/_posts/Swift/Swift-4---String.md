---
title:  String
categories:
- Swift
tags: 
- Swift
hide: true

---


多行字符串以 (""")开头,以(""")结尾.字符串中间的空格不会被忽略

例子:

~~~
func generateQuotation() -> String {
    let quotation = """
        The White Rabbit put on his spectacles.  "Where shall I begin,
        please your Majesty?" he asked.
 
        "Begin at the beginning," the King said gravely, "and go on
        till you come to the end; then stop."
        """
    return quotation
}

摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)”。 iBooks. 
~~~

字符串的增删改查无法像OC一样使用整数下标去获取字符,由于swift String类型采用的是Unicode编码,因此每个字符占用的内存空间可能不一样,所以采用索引(indices)来进行字符串操作

#### 注意
如果索引越界会出发运行时错误,APP会闪退

## 查
index(before:) 
index(after:)
index(_:offsetBy:)

## 增
insert(_:at:)
insert(contentsOf:at:)

## 删
remove(at:)
removeSubrange(_:)

## Substrings
~~~
let greeting = "Hello, world!"
let index = greeting.index(of: ",") ?? greeting.endIndex
let beginning = greeting[..<index]
// beginning is "Hello"
 
// Convert the result to a String for long-term storage.
let newString = String(beginning)

摘录来自: Apple Inc. “The Swift Programming Language (Swift 4)”。 iBooks. 

~~~
Substring不适合长周期存储,因为只要子串被使用,原始字符串就要一直在内存中保持,因此我们需要创建一个新的字符串来保存子串;
