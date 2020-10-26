---
title:  KVC 
date: 2017-12-10 20:07
categories:
- iOS
tags: 
- KVC 
---
# NSKeyValueCoding

一种可以直接通过key的名字来获取对象属性的机制.

## 通览

访问对象值的基本方法是 setValue(_:forKey:),该方法给指定属性设置值,还有value(forKey:)方法,通过指定key获取该属性的value.因此,所有对象属性都可以通过同一种方式去访问.

默认实现依赖于通过对象的访问器方法正常地实现.(或者如果需要的话直接获取实例变量)

## Topics

### 取值

```

//返回指定key的属性值
func value(forKey: String)

//返回指定的key path 推断出的属性值
func value(forKeyPath: String)

//返回一个字典,该字典包含一个数组中包含的所以key对应的属性值
func dictionaryWithValues(forKeys: [String])

/**
 当调用value(forKey:)方法,发现没有key对应属性时,调用此方法.
 
 讨论:
 子类可以重写该方法给未定义的key返回一个可选值,默认实现是会抛出一个NSUndefinedKeyException异常.
 
 */
func value(forUndefineKey: String)


/**
 当key对应对象的属性为一个有序集合时,返回一个可变数组.
 
 */
func mutableArrayValue(forKey: String)

//返回一个可变数组,代理其提供指定key path的序列关系集合的读写权限.
func mutableArrayValue(forKeyPath: String)

//返回一个可变集合,代理其提供指定key的无序关系集合的读写访问
func mutableSetValue(forKey: String)

//返回一个可变集合,代理其提供指定key path的无序关系集合的读写访问.
func mutableSetValue(forKeyPaht: String)

//返回一个可变序列化集合,该集合通过指定key提供对这个特殊的有序集合的读写访问.
func mutableOrderedSetValue(forKey: String)

//返回一个可变序列化集合,该集合通过指定key path 提供对这个特殊有序集合的读写访问.
func mutableOrderedSetValue(forKeyPath: String)

```

### 赋值

```

//给指定keyPath赋值
func setValue(Any?, forKeyPath: String)

//根据指定字典的key对应的value,给key对应的属性赋值
func setValuesForKeys([String : Any])

//当给setValue(_:forKey:)的标量值(比如一个int 或者 float 值)为nil时调用此方法
func setNilValueForKey(String)

/**
 给指定key对应属性赋值

 */
func setValue(Any?, forKey: String)

//当调用setValue(_:forKey:)方法发现没有key对应的属性时,调用此方法
func setValue(Any?, forUndefineKey: String)


```

### 校验

```
//校验一个对象赋值给key对应的属性是否有效,无效则抛出一个错误
func validateValue(AutoreleasingUnsafeMutablePointer<AnyObject?>, forKey: String)

//校验一个对象赋值给对应的keypath属性是否有效,无效抛出一个错误
func validateValue(AutoreleasingUnsafeMutablePointer<AnyObject?>, forKeyPath: String)

```

## 参考资料

[NSKeyValueCoding](https://developer.apple.com/documentation/foundation/object_runtime/nskeyvaluecoding)
