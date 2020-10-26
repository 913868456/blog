---
title:  JavaScriptCore 
date:  2018-02-06 20:26
categories:
- iOS
tags: 
- JavaScriptCore 
---

运行应用程序中的JavaScript程序，并支持在应用程序编写JavaScript。

## 通览
JavaScriptCore框架能够在Swift, Objective-C和C等环境中运行JS脚本.也可以使用JavaScriptCore框架插入自定义的对象到JS脚本中.

## Topics

#### Classes 

###### Class JSContext

一个JSContext对象表示一个JavaScript执行环境.你创建和使用JavaScript contexts去运行来自OC或者Swift代码编写的JavaScript脚本.来访问JavaScript中定义的或计算的值，并使JavaScript能够访问本地对象、方法或函数。

```
//创建一个JavaScript context
//Discussion
//初始化创建context时,会伴随一个新的,独立的virtual machine(一个JSVirtual Machine 对象).您无法在包含在不同virtual machines中的context之间传递JavaScript values(JSValue 对象).创建共享一个virtual machine的context,使用下面的初始化方法
init!()

//创建一个JavaScript context, 与一个具体的virtual machine关联.
//Discussion
//默认情况下,每个context拥有一个独立的virtual machine(一个 JSVirtualMachine对象).您无法在不同的virtual machine中的context传递JavaScript value.使用该方法创建的context,可以和其他的contexts共享它的virtual machine,从而允许在不同的context中传递JSValue对象.
init!(virturalMachine: JSVirturalmachine!)


/**
    执行具体的JavaScript代码
    参数:   需要评估的JavaScript 源码
    返回值: 脚本最后生成的值. 注意,脚本可以导致未定义的JavaScript值
 */
 //Discussion
 //一个脚本运行任何级高级别代码并且给context的全局对象对象添加方法和对象定义.
func evaluateScript(String!)

//执行性具体的JavaScript代码,指定的URL用来对它定位.(调试的时候可以使用)
func evaluateScript(String!, withSourceURL: URL!)

//在一个运行的context中检查回调状态
//返回当前执行JS的context
class func current()

//返回当前执行的JS方法
class func currentCallee()

//返回当前执行JS代码中这个关键字的值
class func currentThis()

//返会从JS代码到当前本地回调的参数
class func currentArguments()

//与Context关联的JS全局对象
var globalObject: JSValue!

//脚本执行中被抛出的异常
var exception: JSValue!

//一个block用来处理JS脚本执行中抛出的异常
var exceptionHandler: ((JSContext?, JSValue?) -> Void)!

//当前context所在的JavaScript virtual machine 
var virtualMachine: JSVirtualMachine!

//context 的描述
var name: String!

//允许通过下标语法,返回在context的全局对象中的具体JS属性值
func objectForKeyedSubscript(Any!)

//允许通过下标语法,给context的全局对象设定指定JS属性
func setObject(Any!, forKeyedSubscript: (NSCoping & NSobject Protocol)!)

//举个例子

JSContext *context = [[JSContext alloc] init];
context[@"makeNSColor"] = ^(NSDictionary *rgb){
    float r = rgb[@"red"].floatValue;
    float g = rgb[@"green"].floatValue;
    float b = rgb[@"blue"].floatValue;
    return [NSColor colorWithRed:(r / 255.f) green:(g / 255.f) blue:(b / 255.f) alpha:1.0];
};

```


###### Class JSManageValue

一个JSManagedValue对象包装了一个JSValue对象.添加"条件引用"行为,以实现对值的自动内存管理.对于一个管理值,主要用例是在一个OC或者Swift对象中存储一个JavaScript值,该OC或者Swift对象本身被导出到JavaScript.

###### Class JSValue

一个JSValue实例是对一个JavaScript 值的参照.您可以使用JSValue类转换JavaScript和objective - c 或者 Swift 之间的基本值(例如数字和字符串)，以便在本地代码和JavaScript代码之间传递数据。您还可以使用这个类创建JavaScript对象，这些对象包装自定义类的本地对象或JavaScript函数，这些函数的实现由本地方法或Block提供。

```
// JS类型与本地类型转换

OC(Swift) Types           JS Types           Notes
-----------------------------------------------------
nil                       undefined 
-----------------------------------------------------
NSNull                    null
-----------------------------------------------------
NSString (String )        String 
-----------------------------------------------------
NSNumber                  Number,Boolean
-----------------------------------------------------
NSDictionary(Dictionary)  Object
-----------------------------------------------------
NSArray (Array)           Array
----------------------------------------------------
NSDate                    Date
-----------------------------------------------------
Object (AnyObject)        Object 

Classs (AnyClass)
-----------------------------------------------------
NSRange,CGRect,           Object          其他结构体类型不支持
CGPoint,CGSize
-----------------------------------------------------
Block(Closure)            Function        JS函数不转换本地block/闭包除非通过一个本地blcok/closure 返回
-----------------------------------------------------

//创建一个JS 值来转换指定的本地对象
init!(object: Any!, in: JSContext!)

//创建一个JS 值来表示指定的布尔值
init!(bool: Bool, in: JSContext!)

//创建一个JS 值来表示指定的Double类型值
init!(double: Double, in: JSContext!)

//创建一个空的JS对象
init!(newObjectIn: JSContext!)

//创建一个空的JS数组
init!(newArrayIn: JSContext!)

//创建一个JSError
init!(neewErrorFromMessage: String!, in: JSContext!)

...

//读取并转换JS值为本地类型
func toObject()

func toObjectOf(AnyClass!)

func toBool()

func toDouble()

func toString()

func toDate()

func toArray()

func toDictionary()

//类型判断

var isUndefined: Bool

var isNull     : Bool

var isBoolean  : Bool

var isNumber   : Bool

...

//JS值比较
func isEqual(to: Any!)

func isEqualWithTypeCoercion(to: Any!)

func isInstance(of: Any!)

//方法调用
func invokeMethod(String!, withArguments: [Any]!)

//容器值
//定义一个JS对象的属性或者修改属性的定义
func defineProperty(String!, descriptor: Any!)

//删除JS对象的属性
func deleteProperty(String!)

//根据下标赋值
func setValue(Any!, at: Int)

//根据下标取值
func atIndex(Int)

//给指定属性赋值
func setValue(Any!, forProperty: String!)

//根据指定属性取值
func forProperty(String!)

//获取持有这个值的context
var context: JSContext!

//通过下标语法获取值
func objectAtIndexedSubscript(Int)

func setObject(Any!, atIndexedSubscript: Int)

func objectForkeyedSubscript(Any!)

func setObject(Any!, forKeyedSubscritp: (NSCoping & NSObjectProtocol)!)

```


###### Class JSVirtualMachine

一个JSVirtualMachine 实例表示JavaScript执行中的一个独立环境.您可以使用该类用于两个主要目的:去支撑并发的JavaScript执行,管理JavaScript与OC,Swift之间桥接过来的对象内存.

###### Protocol JSExport

实现该协议用来导出OC类和他们的实例方法,类方法,属性到JavaScript代码中.

Listing 1  Exporting an Objective-C Class to JavaScript

```
@protocol MyPointExports <JSExport>
@property double x;
@property double y;
- (NSString *)description;
- (instancetype)initWithX:(double)x y:(double)y;
+ (MyPoint *)makePointWithX:(double)x y:(double)y;
@end
 
@interface MyPoint : NSObject <MyPointExports>
- (void)myPrivateMethod;  // Not in the MyPointExports protocol, so not visible to JavaScript code.
@end
 
@implementation MyPoint
// ...
@end
```
Listing 2 Useing an Exported Objectiv-C Class from JavaScript

```
// Objective-C properties become fields.
point.x;
point.x = 10;
// Objective-C instance methods become functions.
point.description();
// Objective-C initializers can be called with constructor syntax.
var p = MyPoint(1, 2);
// Objective-C class methods become functions on the constructor object.
var q = MyPoint.makePointWithXY(0, 0);

```

## 参考文档

[JavaScriptCore](https://developer.apple.com/documentation/javascriptcore?language=occ)
