
---
title:  KVO 
date: 2017-12-17 17:13
categories:
- iOS
tags: 
- KVO 
---
# Key-Value Observing Programming Guide

## 介绍

键值观察是一种机制,该机制允许对象接收其他对象特定属性改变的通知.

> 重要: 为了更好的了解键值观察,您必须理解[键值编码](http://www.jianshu.com/p/7f7360ae8e7b)

一个简单的例子概述了应用中KVO的作用.假设有一个Person对象与一个Account对象相关,表示这个人在银行的存款账户.一个Person实例可能需要知道何时Account实例属性改变对该账户造成影响.比如收支,或者利率.

![示例](http://upload-images.jianshu.io/upload_images/3340896-51f7b5abd6205136.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用KVO,首先确保对象兼容KVO.只要继承自NSObject的对象都是KVO兼容的.然后,必须注册一个观察者Person,观察Account对象实例.Peson发送一个`addObserver:forKeyPath:options:context:`消息给Account.
![注册观察者](http://upload-images.jianshu.io/upload_images/3340896-2ec2849c4a768671.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为了接受来自于Account的改变通知,Peson需要实现`observeValueForKeyPath:ofObject:change:context`方法,所有的观察者都需要实现.一旦注册的KeyPath对应的属性值发生改变,Account将会发送该消息给Person.Person然后基于改变的通知做出相应的响应.

![发送改变通知](http://upload-images.jianshu.io/upload_images/3340896-6518b22626ca9474.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后,当不在需要通知时,在对象销毁之前,使用`removeObser:forKeyPath:` 方法移除观察者.

![移除观察者](http://upload-images.jianshu.io/upload_images/3340896-3eddd2ec631247c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不像使用NSNotificationCenter通知那样,KVO没有一个中心对象给所有的观察者提供改变通知.一旦被观察的对象发生改变通知将会直接发送. NSObject提供了KVO的基本实现.

---

## KVO注册

必须执行以下步骤来使对象接收一个KVO兼容属性的键值观察通知:

- 注册观察者使用`addObserver:forKeyPath:options:context:`
- 在观察者内部实现`observeValueForKeyPath:ofObject:change:context:`方法去接收改变通知消息.
- 当不再需要接收消息时,使用`removeObserver:forKeyPath:`方法取消观察者.至少,在观察者从内存中释放前调用该方法.

> 重要: 不是所有的类的所有属性都是KVO兼容的.

### 注册观察者

一个进行观察的对象首先通过发送一个 `addObserver:forKeyPath:options:context: `消息注册自己和被观察的对象.来传递作为观察者的自身和观察的属性键路径.观察者额外指定一个可选参数和上下文指针用来管理通知方面的内容.

#### Options
可选项参数,使用`|`位操作符来指定多个可选项.影响提供给通知的字典内容,该字典包含观察到的变动信息.并且影响通知的生成方式.

通过指定的`NSKeyValueObservingOptionOld`获取观察的属性改变前的值.通过`NSKeyValueObservingOptionNew`获取改变后新的属性值.

通过`NSKeyValueObservingOptionPrior`可选项,命令观察对象在属性改变前发送一个通知(另外在改变后也发送一个通知).改变信息的字典通过Key为`NSKeyValueChangeNotificationIsPriorKey`, Value为`NSNumber`表示的Yes对象的键值对,标明是一个预改变通知.当需要观察一个属性将要改变时,可以使用该可选项来发送通知.

#### Context

在 `addObserver:forKeyPath:options:context:`消息中的上下文指针包含在相应通知下,传递回的任意数据.可以指定一个NULL类型的数据并且完全依靠键路径字符串去判断改变通知的来源.但是这种方法可能会造成一些问题.如果一个对象的父类出于某种原因也在观察同样的键路径下的属性.

一个更安全和更加可扩展的方法是使用上下文确保你接收的通知目的对象是观察者而不是其父类.
一个类中的特殊命名的静态变量指针可以组成一个好的上下文. **Listing 1** 展示一个为属性观察命名不同上下文的示例.

**Listing 1** 创建上下文指针
```
static void *PersonAccountBalanceContext = &PersonAccountBalanceContext;
statci void *PersonAccountInterestRateContext = &PersonAccountInterestRateContext;

```
Listing 2中的实例论证了一个Person实例如何使用给定的上下文指针注册它自己作为Account实例banlance和interestRate属性的观察者

**Listing 2** 注册属性balance和interestRate的观察者

```

- (void)registerAsObserverForAccount:(Account *)account{
    
    [account addObserver: self forKeyPath:@"balance" options:(NSKeyValueObseringOptionNew | NSKeyValueObservingOptionOld) context: PersonAccountBalanceContext];
    
    [account addObserver: self forKeyPath:@"interestRate" options:(NSKeyValueObservingNew | NSKeyValueObservingOptionOld) context:PersonAccountINterestRateContext];
    
}

```
> 注意:键值观察方法 `addObserver:forKeyPath:options;context:`对观察者,被观察对象,或者上下文,不持有强引用.如果需要的话,应该确保对观察者,被观察对象,和上下文的强引用.

### 接收一个改变的通知

当被观察的属性值发生变化时,观察者会接收到一个`observeValueForKeyPath:ofObject:change:context:` 消息.所有的观察者必须实现该方法.

被观察对象提供触发通知的键路径,它本身作为关联对象,包含更改细节的字典,以及在该键路径下注册的观察者时,提供的上下文指针.

更改内容字典入口`NSKeyValueChangeKindKey`提供发生的更改类型相关信息.如果观察的值已经发生改变,`NSKeyValueChangeKindKey`入口返回`NSKeyValueChangeSetting`.通过依赖注册的观察者指定的可选项.在更改内容字典中`NSKeyValueChangeOldKey`和`NSKeyValueChangeNewKey`包含观察属性改变前和改变后的值.如果属性是一个对象,则直接提供该值.如果属性是一个标量或者结构体,那么对应的值会包装在一个NSValue对象中.

如果观察的属性是一个一对多关系.`NSKeyValueChangeKindKey`入口也会表示是否该集合内的对象被插入,移除,或者替换.分别用`NSKeyValueChangeInsertion`,`NSKeyValueChangeRemoval`,`NSKeyValueChangeReplacement`表示.

更改内容字典入口`NSKeyValueChangeIndexesKey`是一个NSIndexSet对象.明确了在集合中更改的元素的所有下标.如果`NSKeyValueObservingOptionNew`或者`NSKeyValueObservingOptionOld`被指定作为观察者的可选项.那么`NSKeyValueChangeOldKey`和`NSKeyValueChageNewKey`在改变内容字典中用数组来包含关联对象改变前和改变后的值.

在Listing 3中的例子展示了 *Person* 观察者 `observeValueForKeyPath:ofObject:change:context:`方法的实现,并且记录了balance和interestRate属性的改变前后的值.

**Listing 3** `observeValueForKeyPath:ofObject:change:context:`方法实现

```

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context{

    if (context == PersonAccountBalanceContext) {
        
        // Do something with the balance...
        
    }else if (context == PersonAccountInterestRateContext){
        
        // Do something with the interest rate...
        
    }else {
        
        // Any unrecognized context must belong to super
        
        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
        
    }
}

```
如果在注册观察者的时候指定一个NULL上下文,通过比对键路径来判断观察的内容改变情况.如果使用一个单一的上下文给所有观察的键路径,首先测试那个通知的上下文,然后使用键路径来匹配从而判断具体改变的内容.如果给每一个观察的间路径提供一个不同的上下文,正如这里论证的,遗传单一的指针做比较来告诉你是否该通知是发送给这个观察者,并且哪个键路径改变了.

在任何情况下,观察者如果没有识别上下文(或者在一个单一情况下,任何键路径)通常应该调用父类的`obserValueForKeyPath:ofObject:change:context:`实现.因为这意味着父类也注册为观察者去接受通知.

> 注意: 如果一个通知传递到类层级的顶部.NSObject会抛出一个NSInternalInconsistencyException.因为这是一个编程错误: 一个子类没有使用它注册的通知.

### 移除观察者

发送给观察者 removeObserver:forKeyPath:context:消息来移除一个键值观察者.需要指定观察的对象.键路径,和上下文.Listing 4中的例子展示了Person移除自己,作为balance和interestRate的观察者.

**Listing 4** 移除balance和interestRate观察者

```

- (void)unregisteAsObserverForAccount:(Account *)account{
    
    [account removeObserver: self forKeyPath: @"balance" context: PersonAccountBalanceContext];
    
    [account removeObserver: self forKeyPath: @"interestRate" context: PersonAccountInterestRateContext];
}

```

接受到一个 removeObserver:forKeyPath:context: 消息后,观察的对象将不再接受指定的键路径和对象的任何 obserValueForKeyPath:ofObject:change:context:消息 

当正在移除一个观察者时,记住以下几点:

- 如果移除的观察者未注册,则会导致一个NSRangeExcepion异常.调用`removeObserver:forKeyPath:context:`方法一次,则有与之相对应的`addObserver:forKeyPath:options:context:`方法被调用.或者如果不适用的话,把remove方法添加到一个`try/catch` Block中处理潜在的异常.

- 一个观察者在销毁时不会自动移除自身.被观察的对象会持续发送通知,不会顾及到观察者状态的改变.然而,一个改变通知,或者其他消息发送给一个已经释放的对象,会触发一个内存访问异常.因此需要确保观察者释放前一定要移除观察者.

- KVO协议没有提供观察者和被观察对象的访问方式.避免相关的错误发生.一个典型的格式是在观察者初始化的时候(比如在 init 或者 viewDidLoad 方法中),注册为观察者,在delloc方法中取消注册.来确保合适的添加和移除信息.并且确保观察者在内存中移除前已经取消注册.

---
## 注册依赖键

在许多情况下，一个属性的值取决于另一个对象中的一个或多个其他属性的值。如果一个属性的值发生改变,那么派生属性的值也被标记为更改.

### 一对一关系

为了自动触发一对一关系属性的通知,应该重写 `keyPathsForValuesAffectingValueForKey:`方法或者实现一个合适的方法,该方法遵循它定义注册为依赖键的格式.

例如,一个人完整的姓名取决于firs和last names.返回完整姓名的方法可以向下面方法一样写:

```

- (NSString *)fullName {
    
    return [NSString stringWithFormat:@"%@ %@",firstName, lastName];
}

```

当应用观察fullName属性的时候,只要firstName和lastName中任意一个属性发生改变,都必须通知该应用,因为他们影响fullName属性的值.

一种解决办法是重写`keyPathsForValuesAffectingValueForKey:`方法 来指定fullName属性依赖于lastName和firstName属性. **Listing 1**展示了这种依赖的实现.

**Listing 1** `keyPathsForValuesAffectingValueForKey: `方法的实现示例
```
+ (NSSet *)keyPathsForValuesAffectingValueForKey:(NSString *)key {
    
    NSSet *keyPaths = [super keyPathsForValuesAffectingValueForKey:key];
    
    if ([key isEqualToString:@"fullName"]) {
        NSArray *affectingKey = @[@"lastName", @"firstName"];
        keyPaths = [keyPaths setByAddingObjectsFromArray:affectingKeys];
    }
    return keyPaths;
}

```
您的重写通常应该调用super，并返回一个集合，该集合包含了在该集合中产生的任何成员(以便在superclasses中不影响该方法的重写)

通过实现一个类方法也可以实现相同的结果,该类方法遵循命名约定 keyPathsForValuesAffectiing<Key>,该<Key>是依赖属性的名称(首字母大写).按照**Listing 1**中的模式写一个类方法, 如 **Listing 2** 所展示.

**Listing 2** keyPathsForValuesAffecting<Key> 命名约定的实现示例

```
+ (NSSet *)keyPathsForValuesAffectingFullName {
    
    return [NSSet setWithObjects:@"lastName", @"firstName", nil];
}

```
当使用分类给一个存在的类添加计算属性时,不能重写`keyPathsForValuesAffectingValueForKey:`方法.因为在分类中不得重写方法.在那种情况下,实现一个keyPathsForValuesAffecting<Key>类方法可以很好地利用该机制.

> 注意: 一对多关系属性不能使用`keyPathsForValuesAffectingValueForKey:`方法设定依赖.必须观察一对多集合中每个对象的合适的属性并且通过更新依赖键自身来改变它的值.下面内容展示了处理这种情况的一种策略.

### 一对多关系

`keyValuesForValuesAffectingValueForKey: `方法不支持一对多关系属性的键路径.例如,假设有一个department对象,该对象拥有一个一对多关系的属性(employees).employee有一个salary属性.可能该对象希望有一个totalSalary属性来表示employees中所有employee的salary总和.

下面两种情况下的两种可能的解决方案.

1.可以使用键值观察注册父类(本例中是Department)作为所有子类(在本例中是Employees)相关属性的观察者.必须添加和移除作为观察者的父类对象.在`observeValueForKeyPath:ofObject:change:context:`方法中通过更新依赖值来响应变化,就像下面的代码中阐述的一样:

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
 
    if (context == totalSalaryContext) {
        [self updateTotalSalary];
    }
    else
    // deal with other observations and/or invoke super...
}
 
- (void)updateTotalSalary {
    [self setTotalSalary:[self valueForKeyPath:@"employees.@sum.salary"]];
}
 
- (void)setTotalSalary:(NSNumber *)newTotalSalary {
 
    if (totalSalary != newTotalSalary) {
        [self willChangeValueForKey:@"totalSalary"];
        _totalSalary = newTotalSalary;
        [self didChangeValueForKey:@"totalSalary"];
    }
}
 
- (NSNumber *)totalSalary {
    return _totalSalary;
}
```
2.如果您使用的是Core Data，您可以将应用程序的通知中心注册为其托管对象上下文的观察者。托管对象应以类似于键值观察的方式，对子对象发布的相关更改通知作出响应。

---
## 键值观察实现细节

自动键值观察使用一种叫做 isa-swizzling 的技术来实现.

isa 指针,作为建议的名称,指向对象类.该类持有一个分配表.这个分配表包含了实例方法的指针,以及其他数据.

当注册的观察者观察一个对象的某个属性时,被观察对象的isa指针被修改,指向一个媒介类,而不是对象类的真正isa指针.所以isa指针不必映射到实际的类实例.

不应该依靠isa指针来判断类的成员关系.而应该使用类方法去判断一个对象实例的类.

---

# NSKeyValueObserving

一个非正式协议,该协议内容为对象接受其他对象指定属性改变的通知.

## 概述

您可以观察任何对象属性.包括单一属性,一对一关系属性,一对多关系属性.一对多关系属性的观察者被告知改变的属性类型---以及相关的对象.

NSObject提供一个键值观察协议的实现.该实现提供对所有对象的自动观察能力.您可以通过禁用自动观察者通知和使用该协议中的方法实现手动通知来进一步优化通知。

## Topics

### 改变通知

```
/**
 当观察对象的指定键路径下的值发生变化时,通知观察者.
 
 keyPath: 观察对象值已经改变的对应键路径.
 object : 观察对象.
 change : 一个字典用来描述相关对象指定键路径下属性值已经形成的变化.
 context: 提供给注册观察者的值.
 
 讨论:
 对一个对象来说,当它开始发送键路径下值改变通知时,您发送给它一个 addObserver(_:forKeyPath:options:context:)消息,命名应该接受该消息的观察者.当您观察结束时,在观察对象销毁前,您发送给观察对象一个removeObserver(_:forKeyPath:)或者remmoverObserver(_:forKeyPath:context:)消息去取消观察者,并且停止发送改变通知的消息.

 */ 
func observeValue(forKeyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?)

```

### 注册观察

```
/**
 注册观察者接受键路径相关对象的KVO通知消息
  
 observer: 注册KVO通知的对象.观察者必须实现键值观察方法 observeValue(forKeyPath:of:change:context:)
 keyPath: 被观察对象的键路径.该参数不能为nil
 options: 一个在NSKeyValueObservingOptions 值的组合. 
 context: 在obserValue(forKeyPath:of:change:context:)方法中传递给观察者的任意数据
 
 讨论:
 不管是观察者还是被观察对象,引用计数都不会加一.调用该方法的对象必须调用removerObserver(_:forKeyPath:)或者removeObserver(_:forKeyPathL:context:)方法去移除观察者.
 
 */
func addObserver(_ observer: NSObject, forKeyPath keyPath: String, options: NSKeyValueObservingOptions = [], context: UnsafeMutableRawPointer?)

/**
 对一个之前没有注册观察者的对象调用该方法是错误的.
 
 确保注册的观察者在销毁之前调用该方法.

 */
func removeObserver(_ observer: NSObject, forKeyPath keyPath: String)

func removeObserver(NSObject, forKeyPath: String, context: UnsafeMutableRawPointer?)

```

### 通知观察对象的变化

```

func willChangeValue(forKey: String)

func didChangeValue(forKey: String)

//观察对象数组类型时,调用该方法来通知观察对象的变化
func willChange(NSKeyValueChange, valuesAt: IndexSet, forKey: String)

func didChange(NSKeyValueChange, valuesAt: IndexSet, forKey: String)

//观察对象Set类型时,调用该方法来通知观察对象的变化
func willChangeValue(forKey: String, withSetMutation: NSKeyValueSetMutationKind, using: Set<AnyHashable>)

func didChangeValue(forKey: String, withSetMutation: NSKeyValueSetMutationKind, using: Set<AnyHashable>)

```

### 自定义观察

```
//返回一个布尔值标明是否观察对象自动支持KVO.
class func automaticallyNotifiesObservers(forKey: String)

//返回一个键路径集合,该集合内键路径对应的值影响指定key的值.比如计算属性
class func keyPathsForValuesAffectingValue(forKey: String)

//返回一个指针，该指针标识所有在被观察对象注册的观察者的信息。
var observationInfo: UnsafeMutableRawPointer?
```
```
protocol NSKeyValueObservingCustomization

Type methods 

  required
  static func automaticallyNotifiesObservers(for: AnyKeyPath)

  required
  static func keyPathsAffectingValue(for: AnyKeyPath)

Relationships
继承自  NSObjectProtocol
```

### 常量

```

class NSKeyValueObservation

struct NSKeyValueObservedChange

enum NSKeyValueChange{
    
    case setting,
    case insertion,
    case removal,
    case replacement
}

--------------------------------------------------------------
struct NSKeyValueObservingOptions 

#### Constants

    static var new: NSKeyValueObservingOptions

    static var old: NSKeyValueObservingOptions

    static var initial: NSKeyValueObservingOptions

    static var prior: NSkeyValueObservingOptions

--------------------------------------------------------------
struct NSKeyValueChangeKey

enum NSKeyValueSetMutationKind
```

## KVO原理简析
- 当一个object有观察者时，动态创建这个object的类的子类
- 对于每个被观察的property，重写其set方法
- 在重写的set方法中调用` willChangeValueForKey:`和 `didChangeValueForKey:`通知观察者
- 当一个property没有观察者时，删除重写的方法
- 当没有observer观察任何一个property时，删除动态创建的子类

详细的分析可以去看sunyxx的[objc kvo简单探索](http://blog.sunnyxx.com/2014/03/09/objc_kvo_secret/)

## GitHub优秀开源
在项目中使用KVO的时候要时刻谨记移除观察者,否则会抛出异常.这样不经容易出错,而且项目代码看起来也不够漂亮. **facebook**提供了一个很好的解决方案.在该第三方库中,不用再担心移除观察者的问题,代码整体上也比以前更叫干净漂亮.仅需要在项目 **PCH** 文件中 `#import <KVOController/NSObject+FBKVOController.h>`,这样会给每个**NSObject** 对象自动添加KVOController属性,然后直接使用就OK了

```
[self.KVOController observe:clock keyPath:@"date" options:NSKeyValueObservingOptionInitial|NSKeyValueObservingOptionNew action:@selector(updateClockWithDateChange:)];

```
具体使用请看 [KVOController](https://github.com/facebook/KVOController)


# 参考资料
[Key-Value Observing Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177-BCICJDHA)

[NSKeyValueObserving](https://developer.apple.com/documentation/foundation/notifications/nskeyvalueobserving)

[objec kvo简单探索](http://blog.sunnyxx.com/2014/03/09/objc_kvo_secret/)
