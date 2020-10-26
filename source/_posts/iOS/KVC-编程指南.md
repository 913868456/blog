---
title:  KVC 编程指南 
date:  2018-03-15 17:57
categories:
- iOS
tags: 
- KVC 
---
---
## 关于Key-Value Coding

键值编码是一种由NSKeyValueCoding非正式协议启用的机制，协议对象采用该协议来间接访问其属性。当一个对象兼容键值编码时，它的属性通过一个简洁的、统一的消息接口来使用字符串参数寻址。这种间接访问机制补充了实例变量及其相关访问方法所提供的直接访问。

您通常使用访问器方法来获得对对象属性的访问。获取访问器(或getter)返回一个属性的值。设置访问器(或setter)设置属性的值。在objective-c中，还可以直接访问属性的底层实例变量。在任何一种方法中访问对象属性都很简单，但需要调用属性特定的方法或变量名。随着属性列表的增长或变化，也必须使用访问这些属性的代码。与此相反，一个键值编码兼容的对象提供了一个简单的消息接口，它在所有属性中都是一致的。

键值编码是许多其他Cocoa技术的基础概念，如键值观察、Cocoa绑定、Core Data和applescript能力。在某些情况下，键值编码也有助于简化代码。

### 使用服从Key-Value Coding 的对象

对象通常在继承NSObject(直接或间接)时即采用了键值编码，它们都采用NSKeyValueCoding协议，并为基本方法提供默认实现。这样的对象通过一个紧凑的消息传递接口使其他对象能够执行以下操作:

- 访问对象的属性。该协议指定了一些方法，例如通用的getter valueForKey:和通用的setter setValue:forKey:用于访问对象属性通过属性名(或键，参数化的字符串)。这些对象相关方法的默认实现使用键来定位和与底层数据交互。

- 操作集合属性。与其他属性一样，访问方法的默认实现与对象的集合属性(例如NSArray对象)实现相同。此外，如果一个对象定义了一个属性的集合访问器方法，那么它可以使用键值访问获取到集合的内容。这通常比直接访问更有效，并允许您通过标准化接口使用自定义集合对象。

- 在集合对象上调用集合操作符。当在键值编码兼容对象中访问集合属性时，可以将集合操作符插入到关键字字符串。集合操作符指示默认的NSKeyValueCoding getter实现对集合的取值操作，然后返回一个新的、经过筛选的集合版本，或者一个表示集合特征的单一值。

- 访问非对象属性。协议的默认实现会检测非对象属性，包括标量和结构体，并自动将它们封装起来，作为在协议接口上使用的对象。此外，该协议声明了一种方法，允许一个服从协议的对象在通过键值编码给一个非对象类型赋值为nil时,提供一个合适的响应.

- 通过keyPath访问属性。当您有一个符合键值编码的对象的层级结构时，您可以使用基于keyPath的方法在层次结构中调用单个方法来获取或来设置一个值。


### 为对象采用键值编码

为了使您自己的对象具有键值编码兼容，您确保他们采用了`NSKeyValueCoding`非正式协议并实现了相应的方法，例如`valueForKey:`作为通用的getter和 `setValue:forKey:` 作为通用的setter。幸运的是，如上所述，NSObject采用了这个协议，并为这些和其他基本方法提供了默认实现。因此，如果您从NSObject(或它的许多子类)派生对象，那么大部分工作已经为您完成了。

为了使默认的方法能够执行它们的工作，您可以确保对象的访问器方法和实例变量遵循某些定义良好的格式。这允许默认实现在对键值编码消息的响应中找到对象的属性。然后，您通过提供的校验方法可选地去扩展和自定义键值编码并且处理某些特殊的情况.

### Swift中的键值编码

从NSObject继承的Swift对象或其子类的默认属性遵循键值编码协议。而在Objective - C中，属性的访问器和实例变量必须遵循特定的模式，Swift的标准属性声明会自动保证这一点。另一方面，许多协议的特性要么不相关，要么使用在objective - c中不存在的本地Swift构造或一些技术能够更好地处理。例如,所有的Swift属性都是对象,你不必再操心对非对象属性默认实现的特殊处理

因此，虽然键值编码协议方法直接转换为Swift，但这个指南主要关注objective - c，在这里您需要做更多的工作以确保遵从性，在什么地方键值编码通常是最适合的。

### 其他依赖键值编码的 Cocoa 技术

一个符合键值编码的对象在Cocoa技术参与广泛，这些技术依赖于这种访问方式，包括:

- 键-值观察。这个机制使对象能够注册异步通知，这是由另一个对象属性的变化所驱动的，如键值观察编程指南所述。

- Cocoa 绑定。这个技术集合完全实现了一个模型-视图-控制器范例，模型封装应用程序数据，视图显示和编辑数据，控制器在两者之间进行协调。阅读Cocoa绑定编程主题，了解更多关于Cocoa绑定的知识。

- Core Data。这个框架为对象生命周期和对象持久化相关的常见任务提供了通用的和自动化的解决方案。更多了解您可以阅读Core Data编程指南。

- AppleScript。这种脚本语言可以直接控制脚本应用程序和macOS的许多部分。Cocoa的脚本支持利用了键值编码来获取和设置脚本对象中的信息。NSScriptKeyValueCoding非正式协议中的方法为使用键值编码提供了额外的功能，包括通过索引在多值密钥中获取和设置键值，并强制(或转换)键值到适当的数据类型。AppleScript概述提供了AppleScript及其相关技术的高级概述。

---

## 键值编码原理

### 访问对象的属性

一个对象在它的接口声明中明确属性.这些属性属于下面类别中的一种:

- 属性(Attributes).这些属性是一些单一的值,例如一个标量,字符串,或者布尔类型的值.值对象诸如NSNumber和其他不可变类型比如NSColor也都被认为是属性.
- 一对一关系.这些属性是可变对象并且拥有他们自己的属性.一个对象的属性可以改变而无需改变对象自身.举例来说,一个 `bank account` 对象可能有一个`owner`属性,该属性是一个`Person`类型的对象.而`person`对象又有一个`address`属性.`owner`的`address`可能改变而 `bank account` 却不需要改变对`owner`的引用持有.`bank account` 的` owner`没有改变,仅仅是它的地址改变了.

- 一对多关系.这些属性是集合类型对象.尽管自定义集合类页可以实现,但是通常用一个NSArray或NSSet实例去持有该集合.

BankAccount对象声明每种属性类型,如下:

**Listing 2-1** ` BankAccount`对象的属性

```

@interface BankAccount : NSObject

@property (nonatomic) NSNumber* currentBalance;   //An Attribute 
@property (nonatomic) Person*   owner;            //A to-one relation  一对一关系
@property (nonatomic) NSArray< Transaction* >* transactions;//A to-many relation 一对多关系

@end

```

为了持有包装的属性,一个对象在它的接口中为其属性提供了访问器方法.对象的创建者可以自定义访问器方法,或者依赖编译器去自动生成访问器方法.总之,代码的创建者使用这些访问器必须在编译前写明属性的名字.访问器方法的名字存储在静态存储区.举例来说,给定的 `bank account` 对象声明如** Listing 2-1 **,编译器会为`myAccount`对象生成一个setter方法来供你使用:

```

[myAccount setCurrentBalance:@(100.0)];

```

这种赋值方式直接,但是缺乏灵活性. 一个键值编码键值对象,提供一个更普遍的机制,使用字符串标签去访问一个对象的属性.

#### 使用key和key Path来识别一个对象的属性

key 是一个字符串用来识别具体的属性.通俗地讲, key代表一个属性.key必须使用ASCII编码,不可用包含空格,并且通常以小写字母开头(尽管有例外,比如在许多类中的URL属性)

因为`BankAccount`类兼容键值编码,它识别keys所对应的`owner`, `currentBalance`, `transactions`,这些对象的属性名.而不是调用 `setCurrentBalance: `方法,可以根据它的key来给其属性赋值:

```

[myAccount setValue:@(100.0) forKey:@"currentBalance"];

```

事实上,你可以给myAccount对象使用该方法设置所有属性的值.只需要替换不同的key参数就行.因为参数是一个字符串,使其可以作为一个变量在运行时中进行操作.

key path 是一个以.符号做为key分割标记,来遍历对象的属性.在遍历中第一个key对应的属性跟receiver关联.每个子遍历key都更它上一级的属性值关联.key path对于多层次结构对象的属性调用很有用.

举例来说,keypath owner.address.street 应用于一个bank account实例对象.对应关联一个street字符串作为其值.假设Person和Address类也兼容键值编码,该字符串存储在bank account的owner属性所对应的address中.

> 注意
在 Swift 中, 你可以使用#keyPath表示方式来取代使用一个字符串去表示一个key或者keypath.这种方式提供了编译时检测的优点.

#### 使用key获取属性值

一个兼容键值编码的对象,该对象继承自NSObject,一个从NSObject继承的对象，它提供了协议的基本方法的默认实现，它会自动采用该协议，并带有某些默认行为。这样的对象至少实现了以下基本的key-bassed getters:

- `valueForKey: `根据key参数返回属性的值.如果以key命名的属性根据 访问器搜索格式 中的规则无法找到,该对象会给自己发送一个 `valueForUndefinedKey:` 消息. `valueForUndefinedKey: `方法的默认实现会吊起一个`NSUndefinedKeyException`,但是子类可以重写,并且更优雅的处理这种情况.

- `valueForKeyPath:` 返回对象指定键路径下的值.任何在键路径遍历下没有与之相匹配的键值编码兼容,意味着使用 `valueForKey: `的默认实现无法找到相对应的访问器方法.对象则调用 `valueForUndefinedKey:` 方法.

- `dictionaryWithValuesForKeys:` 根据数组里面所有key返回相关的键值字典.数组中每个key都会调用` valueForKey: `方法.返回值为一个字典,包含数组中所有的key和与之相关的值.

> 注意
集合类型对象,比如NSArray,NSSet,NSDictionary,不能把nil作为值包含进去.可以使用NSNull对象代表一个空值.NSNull提供一个实例来表示对象属性的空值. `dictionaryWithValuesForKeys: `方法和相关的 `setValuesForKeysWithDictionary: `方法 的默认实现 自动在NSNull(在字典参数中)和nil(在存储属性中)之间转换

当你使用一个键路径去寻址一个属性时,如过键路径下的最后的key对应的是一个一对多的属性(意味着关联一个集合),返回值是一个包含所有键对应的值的集合.举例来讲,请求` transactions.payee `路径下的值,返回在所有`transactions`下`payee`对象的数组.keypath中多个数组同样适用.`accounts.transactions.payee`返回在所有`accountes`对象下,所有`transactions`下的`payee`组成的数组.


#### 使用key给属性赋值

和getter方法一样,键值编码对象也提供了基于NSKeyValueCoding协议实现的一组setter方法:


- `setValue: forkey:` 给对象指定键赋值.该方法默认自动解包表示标量和结构体的NSNumber和NSValue对象,并且将这些解包后的值赋给对应属性.
如果将要赋值的键不存在,则对象调用 `setValue: forUndefinedKey:` 方法.默认会吊起`NSUndefinedKeyException.`然而,子类可以重写该方法,并且以自定义的方式处理.

- `setValue: forKeyPath:` 设置指定键路径下的值.对象在键路径遍历下未找到相关属性,则调用`setValue:forUndefinedKey:`方法.

- `setValuesForKeyswithDictionary: `给字典中所有key对应的属性赋值.默认实现是给每个键值对调用`setValue:forKey:`方法.如果有必要的话,用NSNull对象替换nil

在默认实现中,当试图给一个非对象属性赋值nil时,键值编码兼容对象会给自身发送一个`setNilValueForKey:`消息.该方法默认会吊起`NSInvalidArgumentException`.但是一个对象可以重写该方法,去替代默认实现或者用一个标量值替代.


#### 使用键去简化对象访问

通过键值编码的setter和getter能够简化代码.在macOS中,NSTableView,NSOutlineView对象在他们的列中标记一个字符串id.如果模型对应的表格不兼容键值编码,表格数据源方法会去强制检查每一列的id找到相关属性并返回.未来,当添加一个属性给模型的时候,例如Person对象的那中情况,你必须重新访问数据源方法,为新属性的测试添加另一种条件,并且返回相关的值.

**Listing 2-2** 不适用键值编码的数据源方法实现

```
- (id)tableView:(NSTableView *)tableview objectValueForTableColumn:(id)column row:(NSInteger)row
{
    id result = nil;
    Person *person = [self.people objectAtIndex:row];
 
    if ([[column identifier] isEqualToString:@"name"]) {
        result = [person name];
    } else if ([[column identifier] isEqualToString:@"age"]) {
        result = @([person age]);  // Wrap age, a scalar, as an NSNumber
    } else if ([[column identifier] isEqualToString:@"favoriteColor"]) {
        result = [person favoriteColor];
    } // And so on...
 
    return result;
}


```

**Listing 2-3** 展示了一个使用兼容键值编码的`Person`对象 更紧凑的数据源方法实现方式. 只需要使用 `valueForKey` getter方法,数据源方法就会使用列的标签作为key返回合适的值. 除了更简洁外,也更加通俗易懂,因为只要列标识符总是与模型对象的属性名匹配，它就会在稍后添加新列时保持不变。

**Listing 2-3** 使用键值编码实现数据源方法

```

- (id)tableView:(NSTableView *)tableview objectValueForTableColumn:(id)column row:(NSInteger)row
{
    return [[self.people objectAtIndex:row] valueForKey:[column identifier]];
}

```

---
### 访问集合属性

键值编码兼容对象像公开其他属性的方式一样公开它的一对多属性.可以使用` valueForKey: `和 `setValue: forKey:` (或者他们的 keyPath) 方法来给对象的集合属性赋值和访问. 然而,当想要对集合内容进行操作时,通常使用协议定义的可变代理方法是最有效的.

协议定义了三个不同代理方法用来对集合对象进行访问:

- `mutableArrayValueForKey:` 和 `mutableArrayValueForKeyPath:`

方法返回一个代理对象,该对象是一个`NSMutableArray`对象.

- `mutableSetValueForKey:` 和 `mutableSetValueForKeyPath:`

方法返回一个代理对象,该对象是一个`NSMutableSet`对象.

- `mutableOrderedSetValueForKey:` 和 `mutableOrderedSetValueForKeyPath:`

方法返回一个代理对象,该对象是一个`NSMutableOrderedSet`对象.

当操作一个代理对象时,从集合中添加.删除,替换对象,协议的默认实现会相应地修改对应的下标属性.这种方式比使用 valueForKey: 方法获取不可变集合效率更高,创建一个可变集合,然后使用setValue: forKey: 消息将元素存储到集合中. 在许多情况下, 与直接使用可变属性相比,这种方式效率更高.这些方法为集合中对象的键值观察提供了额外的好处.

---

### 使用集合运算符

当发送一个`valueForKeyPaht:`消息给键值兼容对象时,可以在key path 中嵌入一个集合运算符,集合运算符是关键字列表中的一个.在它前面有一个@符号,来表示在getter返回前,以某种方式执行数据操作. valueForKeyPath:的默认实现由NSObject来提供.
当一个 keyPath 包含一个集合运算符时,运算符前面的keyPath,称为left key path,表示该集合相对于消息接受者的操作. 如果将消息直接发送到集合对象,例如NSArray实例,则可以省略左边的key path.
运算符后面的keypath部分,称为right key Path,指定运算符要操作的集合中的对象属性.处理@count所有的集合运算符都需要一个right key path. Figure 4-1 阐明了运算符keypath格式.

下图为运算符 key path  格式

![Figure 4-1](http://upload-images.jianshu.io/upload_images/3340896-63cef07595f087a5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

集合运算符的三种类型:
Aggregaion Operators, Array Operatiors, Nesting Operators

下面这段代码演示了如何执行每个运算符的相关操作.
```
@interface Transaction : NSObject
 
@property (nonatomic) NSString* payee;   // To whom
@property (nonatomic) NSNumber* amount;  // How much
@property (nonatomic) NSDate* date;      // When
 
@end

```

#### Aggregation Operators

@avg

当使用@avg运算符时,`valueForKeyPaht:`会根据Right key Path中指定的属性读取集合中每个对象相关的值.并且计算他们的平均值.然后会返回一个包含结果的NSNumber实例.

下面代码,可以获取集合中所有转账的平均值

```

NSNumber *transactionAverage = [self.transactions valueForKeyPath:@"@avg.amount"];

```

@count

使用@count 运算符, `valueForKeyPath: `返回一个NSNumber实例表示集合中对象的数量. right key path忽略.

获取转账的笔数:

```

NSNumber *numberOfTransactions = [self.transactions valueForKeyPath:@"@count"];

```

@max

当使用@max运算符时, 返回集合中right key path 对应属性的最大值. 内部是使用 compare:方法比较集合中该属性对应的值.在搜索right key path对应属性值时,会忽略集合中的nil值.

获取日期值的最大值,表示最近的转账日期.

```

NSDate *latestDate = [self.transactions valueForKeyPath:@"@max.date"];

```

@min

使用@min 运算符时, `valueForKeyPath: `方法搜索集合中righ key path 对应属的值.并且返回最小的一个.

返回转账日期中最早的日期.

```

NSDate *earliestDate = [self.transactions valueForKeyPath:@"@min.date"];

```

@sum

使用 @sum 运算符,计算集合中所有keypath对应属性值的总和.

```

NSNumber *amountSum = [self.transacitons valueForKeyPath:@"@sum.amount"];

```

#### Array Operators 

Array 运算符使用valueForkeyPath: 返回right key path 对应的对象集合.

> 重要
如果使用数组运算符时,任何支对象是空值的话,valueForKeyPath: 方法会抛出异常,

@distinctUnionOfObjects

使用@distinctUnionOfObjects 运算符, valueForKeyPath: 方法会创建并返回一个数组,该数组包含集合中对象的并集,这些对象与right key path 指定的属性对应.

获取 payee 属性值的集合.

```

NSArray *distincPayees = [self.transactions valueForKeyPath:@"@distinctUnionOfObjects.payee"];

```

> 注意
@unionOfObjects 运算符提供相似的行为,但是不会移除重复的对象.

@unionOfObjects

```
NSArray *payees = [self.transactions valueForKeyPath:@"@unionOfObjects.payee"];

```


#### Nesting Operators

嵌套运算符在嵌套集合中使用.每个集合入口自身便包含一个集合.

> 重要

如果使用嵌套运算符时,任何支对象是nil的话,valueForKeyPath:方法会抛出一个异常.

下面的arrayOfArrays是一个嵌套数组

```

NSArray * moreTransactions = @[<# transacion data #>];
NSArray * arrayOfArrays    = @[self.transactions, moreTransactions];

```

@distinctUnionOfArrays

使用该运算符时,返回一个数组,该数组包含所有集合中指定key path下所有属性对应的对象.

```

NSArray *collectedDistinctPayees = [arrayOfArrays valueForKeyPath:@"@distinctUnionOfArrays.payee"];

```

> 注意
@unionOfArrays 运算符与@distictUnionOfArrays运算符功能类似,但是不会移除重复对象.


@unionOfArrays

```

NSArray *collectedPayees = [arrayOfArrays valueForKeyPath:@"unionOfArrays.payee"];

```

@distincUnionOfSets

与@distincUnionOfArrays类似,假如示例数据是以集合存储而不是数组,则返回的是一个NSSet集合对象.

---

### 表示非对象类型值 

NSObject提供的键值编码协议方法的默认实现不仅适用于对象属性,在非对象属性中同样适用. 默认实现在对象参数,返回值,和非对象属性之间自动切换.这将允许基于key-based 签名的getter和setter方法保持一致,即使存储属性是一个标量或者结构体.

> 注意
在Swift中所有属性是对象类型.本节只在OC属性中适用.

当调用协议中的getter方法时,比如 valueForKey:, 默认实现决定调用某个特殊的访问器方法或者为指定key提供值的实例变量.如果返回值不是一个对象,getter使用该值去创建一个NSNumber对象(对于标量) 或者NSValue对象(对于结构体) 并且 返回NSNumber或者NSValue.

类似的,默认情况下,setter方法例如`setValue:forKey: `通过一个属性访问器或者实例变量来决定其数据类型.如果属性的数据类型不是对象类型,setter方法首先发送一个合适的`<type>Value `消息给传入的值对象,来提取下标数据,并且将设置的值存储进去.

> 注意
当使用键值编码协议中的一个setter方法给一个非对象属性赋空值时,setter没有明显的执行行为.因而,它会给接受setter方法的对象发送一个`setNilValueForKey:`方法.默认会吊起一个`NSInvalidArgumentException`异常.但是子类可以重写改方法.

#### 包装和解包标量类型

标量类型的默认键值编码实现是使用一个NSNumber实例包装起来,对于每种数据类型,使用下标属性的值创建一个NSNumber对象来提供一个getter返回值.同样该表也展示了使用的访问器方法,该方法是在一个赋值操作中,从setter输入参数中提取值.

标量类型被包装在NSNumber对象中.

> 注意
在macOS中, 由于历史原因, BOOL的类型定义是作为signed char类型.KVC不会区分这些.所以,当key是一个BOLL类型值时,使用`setValue:forKey:`方法不能传递像@"true"或者@"YES"这样的值,KVC会尝试执行charValue(应为BOOL类型继承自char类型),但是NSString不会实现该方法,那样会导致一个运行时错误. 反而,只能传递一个NSNumber对象,比如@(1)或者@(YES),当Key时一个BOOL类型时,做为`setValue:forKey:`的参数.这种限制在iOS中不适用,BOOL类型被定义为本地Boolean类型bool并且KVC执行`boolValue`方法.这种设置值的方式对一个NSNumber对象或者一个合适格式的NSString对象都是适用的.


#### 包装和解包结构体类型

自动包装和解包不限于NSPoint,NSRange,NSSize,NSRect.结构体类型(OC编码的)都可以包装在一个NSValue对象中.

**Listing 5-1** 使用一个自定义的结构体的示例类

```
typedef struct {
    
    float x, y, z;
    
}ThreeFloats;

@interface MyClass

@property (nonatomic) ThreeFloats threeFloats;

@end

```
使用该类的一个叫做myClass的实例,使用KVC获取threeFloats属性的值.

```
NSValue *result = [myClass valueForKey:@:"threeFloats"];

```
`valueForKey:`方法的默认实现调用`threeFloats`的getter方法.然后返回返回一个NSValue对象,该对象包装着返回值.

类似的,也可以使用KVC给treeFloats属性赋值

```
ThreeFloats floats = {1.,2.,3.};

NSValue * value = [NSValue valueWithBytes:&floats objCType:@encode(ThreeFloats)];

[myClass setValue:value forKey:@"ThreeFloats"];

```

默认实现使用一个 `getValue:`消息来解包数据值,然后使用解包的结构体作为参数调用`setThreeFloats:`方法.


#### 校验属性

键值编码协议定义了支持属性校验的方法.使用键值编码兼容对象,可以读取和写入属性数据.也可以使用通过一个key或者keyPath来校验一个属性.当调用` validateValue:forKey:error:`(或者 `validateValue:forKeyPath:error`)方法时, 协议的默认实现会为找到一个匹配格式` validate<key>:error:`的方法而去搜索接受校验信息的的对象,.如果对象没有这样的方法,默认校验成功,返回YES.当一个指定的属性校验方法存在时,返回该方法的调用结果.

> 注意
仅在OC中使用校验.在swift中,属性校验依赖于可选类型和强引用类型检查的编译器来实现.

因为属性指定校验方法通过引用接收值和错误参数.校验可能会有三种结果:

1.校验方法认为值对象有效,并且不改变值和错误参数,返回YES.
2.校验方法认为值对象无效,但是选择不改变值对象.在这种情况下,返回值为NO并且给NSErro对象设置一个错误引用来说明失败原因.
3.校验方法认为值对象无效,但是创建了一个新的,有效的值替换掉原来的值.在这种情况下,方法返回YES,而将错误对象保持不变。在返回之前，该方法修改了指向新值对象的值引用。当它进行修改时，方法总是创建一个新的对象，而不是修改旧的对象，即使值对象是可变的。

**Listing 6-1** 属性名校验

```

Person * person = [[Person alloc] init];

NSError * error;

NSString *name = @"John";

if(![person validateValue:&name forKey:@"name" error:&error]){
    
    NSLog(@"%@",error);
}

```

#### 自动校验
一般来讲,不管是键值编码协议还是对象的默认实现都没有定义自动执行校验的机制.在您的app需要该功能时,使用该方法校验.

### 访问器搜索模式

#### 基础 Getter 搜索模式

`valueForKey:`方法的默认实现,给定一个key作为输入参数,执行下面流程,从接受返回值的对象进行操作.

1.搜索访问器方法,根据 `get<KEY>`, `<key>`, `is<Key>`或者`_<key>`这样的名称和顺序来搜索.如果发现,调用并且执行步骤5返回结果.否则执行下一步.

2.如果没有一个访问器被发现,搜索匹配格式为`countOf<Key>`和`objectIn<Key>AtIndex:`和`<key>AtIndexes:`这些方法.
如果这些方法中的第一个和剩余两个方法中至少一个被找到.创建一个集合代理对象,该对象响应所有的NSArray方法,并且返回该集合对象.否则,执行步骤3.

3.如果没有一个访问器或者数组访问方法被找到,那么接着找 `countOf<key>`, `enumeratorOf<Key>`,`memberOf<Key>:` 这三个方法.

如果所有三个方法都被找到,创建一个集合代理对象,该对象的所有方法响应NSSet的所有方法,并且返回该对象.否则,执行步骤4

4.如果没有一个访问器方法或者组合访问方法被找到,并且接受对象的类方法 `accessInstanceVariablesDirectly`返回YES,接着按照顺序搜索名字为`_<key>`,`_is<Key>`,或者`is<Key>`
的实例变量,如果找到,直接获取该实例变量的值,执行步骤5,否则执行步骤6

5.如果检索到的属性值是一个对象指针,返回该结果.
如果该值是一个NSNumber支持的标量类型,将它存储在一个NSNumber实例中,并且返回该实例.
如果结果是一个NSNumber不支持的标量类型,将其转换为NSValue对象并返回该对象.

6.如果以上所有的步骤都失败,调用`valueForUndefinedKey:`方法,默认吊起一个异常,但是子类重写.


#### 基础 Setter 搜索模式

`setValue:forKey:`方法执行下面流程:

1. 按照顺序set<Key>:, _set<Key>需要第一访问器.如果找到,使用输入值(或者未包装的值),执行该方法,结束.

2. 如果没有访问器被找到,并且类方法 `accessInstanceVariablesDirectly`返回YES,按照名字为_<key>,_is<Key>,<key>,is<Key>的顺序寻找
实例变量.如果找到,则给实例变量直接赋值,结束.

3.以上方法未找到访问器或实例变量,执行`setValue:forUndefinedKey:`方法,默认抛出异常,子类可以重写该方法.


Array 和 Set 的搜索格式与以上类似,不赘述.

---

想要更多地了解关于 `KVC` 的API,请移步 [KVC](https://www.jianshu.com/p/7f7360ae8e7b)

## 参考资料
[Key-Value Coding Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueCoding/index.html#//apple_ref/doc/uid/10000107-SW1)
