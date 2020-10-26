
---
title:  协议
categories:
- Swift
tags: 
- Swift
hide: true
---

#### 定义
```
protocol SomeProtocol {
    // protocol definition goes here
}
```

#### 遵循协议

```
class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
    // class definition goes here
}
```
#### 属性要求

```
protocol SomeProtocol {
    var mustBeSettable: Int { get set }
    var doesNotNeedToBeSettable: Int { get }
}
```
#### 类属性

属性前缀添加关键字"static",协议方法与此同理,func前面添加"static"

```
protocol AnotherProtocol {
    static var someTypeProperty: Int { get set }
}
```
#### Mutating Method Requirement

在方法前添加 mutating 关键字,可在该方法中修改实例的属性.只能在枚举和结构体中使用.

```
enum OnOffSwitch: Togglable {
    case off, on
    mutating func toggle() {
        switch self {
        case .off:
            self = .on
        case .on:
            self = .off
        }
    }
}
var lightSwitch = OnOffSwitch.off
lightSwitch.toggle()
// lightSwitch is now equal to .on
```
