---
title:   Objective-C Runtime
date: 2017-11-20 21:25
categories:
- iOS 
tags: 
- runtime
---
描述macOS OC运行时库支持的函数和数据结构.

## 通览

OC运行时是一个运行时库,该库用来支持OC语言的动态属性.并且这种情况对所有的的Objectiv-C的app都有关系.OC运行时库支持的函数实现在 /usr/lib/libobjc.A.dylib中的分享库中可以查到.(自己测试是在资源库文件夹下面)

在OC编程中,您完全没有必要直接地使用OC运行时库.这些Api的主要作用是作为OC和其他语言的桥接层.或者更底层的调试.

OC运行时库在macOS 中的实现是特殊的.对于其他平台,GNU编译集合使用类似的Api提供了一种不同的的实现.这个文档只包含了macOS的实现.

底层的OC运行时Api在OS X 版本10.5中做了显著更新.许多函数和所有现有的数据结构被替换为新的函数.老的函数和数据结构在32位模式下被弃用.64位模式下仍然存在.Api在64位模式下约束了一些在32位模式下表示的整型数据值.比如 class Count, protocol Count, methods per class, ivars per class, arguments per method, sizeof(all arguments) per method, class version number.另外,新的Objective-C ABI 对32位模式下的对象实例化约束更多.并且三个其他的24位模式下表示的一些值 methods per class, ivars per class, sizeof(a single ivar).最后,
老式的NXHashTable和NXMapTable被限制到四百万项.

> 字符串编码
> 在运行时API中所有的字符指针应该
 被看作UTF-8编码格式.

## 该文档适用哪些人?

这个文档适用那些对OC运行时有兴趣学习的读者.
因为这不是关于C的文档.该文档仅当做对OC开发者的一些拓展.

## Topics 

### 与Class一起使用

```

    /*
     cls: 一个类对象
     返回值: 类的名称,如果传Nil则为空字符串
     **/

    func class_getName(_ cls:AnyClass?) -> UnsafePointer<Int8>
    
    /*
     cls: 一个类对象
     返回值: 该类的父类. 如果cls为根类或者cls为Nil的话,返回值为Nil
     
     讨论:
     你应该通常使用NSObject的 superclass()方法替换该方法
     **/

    func class_getSuperclass(_ cls: AnyClass?) -> AnyClass?
    
    /*
     返回一个布尔值, 该值表示是否一个类对象是一个元类
     
     cls: 一个类对象
     返回值: 如果参数是一个元类,则返回true. 如果非元类,则返回false,或者参数为Nil也返回false.
    
     **/

    func class_isMetaClass(_ cls: AnyClass?) -> Bool

    /*
     返回一个类的实例占用内存的大小
     
     cls: 一个类对象
     返回值: 类实例的占用的字节.如果参数为Nil则返回0
    **/

    func class_getInstanceSize(_ cls: AnyClass?) -> Int
    
    
    /*
     返回给定类的一个指定实例变量
     
     cls: 想要获取实例变量的类
     name: 去获取的实例变量名
     返回值: 指定名字的实例变量的指针
    **/

    func class_getInstanceVariable(_ cls: AnyClass?, _ name: UnsafePointer<Int8>) -> Ivar?
    
    /*
     返回某个类指定名字的类变量
     
     cls: 要获取类变量的类
     name: 要获取的类变量名称
     返回值: 指定名字的类变量的指针
    **/

    func class_getClassVariable(_ cls: AnyClass?, _ name: UnsagePointer<Int8>) ->Ivar?
    
    /*
     给一个类添加一个实例变量
     
     返回值: 如果实例变量加入成功,返回true.否则返回false(比如该类已经包含一个相同名字的实例变量)
     
     讨论:
     这个函数只能在objc_allocateClassPair(_:_:_:)之后调用.在objc_registerClassPair(_:)之前调用.不支持给一个已经注册过的类添加实例变量.
     
     操作的类不能使元类.不支持给一个元类添加实例变量.
     
     实例变量以字节的最小对齐方式为1<<align.实例变量的最小对齐方式取决于实例变量的类型和设备的架构.对于任意指针类型的变量,传log2(sizeof(pointer_type))
    
     **/

    func class_addIvar(_ cls: AnyClass?,_ name: UnsagePointer<Int8>, _ size: Int, _ alignment: UInt8, _ types: UnsagePointer<Int8>?) -> Bool
    
    /*
     获取类的实例变量列表
     
     cls: 需要拷贝属性列表的类
     outCount: 在返回时,包含返回数组的长度.如果outCount是NULL,则不反悔该长度.
     返回值: 一个指针数组描述类声明的实例变量.任何被父类声明的实例变量不包含在内.数组包含* outCount指针，后跟一个空终止符。您必须使用free()函数释放数组.
     如果类声明中没有实例变量.或者cls为Nil,则返回NULL 并且*coutCount指针为0
     **/

    func class_copyIvarList(_ cls: AnyClass?, _ outCount: UnsafeMutablePointer<UInt32>?) -> UnsafeMutablePointer<Ivar>?
    
    //返回指定类实例变量布局的描述
    func class_getIvarLayout(AnyClass?)
    
    //设置指定类实例变量布局
    func class_setIvarLayout(AnyClass?, UnsafePointer<UInt8>?)
    
    //返回弱引用实例变量的布局描述
    func class_getWeakIvarLayout(AnyClass?)
    
    //设置弱引用实例变量的布局
    func class_setWeakIvarLayout(AnyClas?, UnsafePointer<UInt8>?)
    
    /*
     返回指定类的指定名称的属性
     
     返回值: 一个objc_property_t类型的指针.如果类中并未声明该名称属性或者cls为Nil则返回NULL.
     **/
    func class_getProperty(_ cls: AnyClass?, _ name: UnsafePointer<Int8>) -> objc_property_t?
    
    /*
     描述声明的一个类的属性
     
     cls: 想要检查的类
     outCount: 在返回时,包含返回数组的长度.如果outCount 是NULL,数组长度不返回.
     返回值: objc_property_t类型的指针数组.任何父类声明的属性不包含在内.数组包含* outCount指针，后跟一个空终止符。您必须使用free()函数释放数组.
     如果类声明中没有实例变量.或者cls为Nil,则返回NULL 并且*coutCount指针为0
     **/

    func class_copyPropertyList(_ cls: AnyClass?, _ outCount: UnsageMutablePointer<UInt32>?) -> UnsafeMutablePointer<objc_property_t>?
    
    /*添加一个新方法给指定类.指定方法名和方法实现
    
      cls:需要添加方法的类 
      name: 指定添加的方法名  
      imp: 方法的实现的函数,该函数必须携带两个参数 self 和  _cmd
      types: 一个字符数组,用来描述方法的参数类型.参考苹果官方文档 [Type Encodings](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100)
      返回值: 如果添加成功,返回true 否则返回false(例如添加的方法已经存在)
      
      讨论:
      该方法会重载父类的实现,但是并不会替换在这个类中已经存在的实现.想要改变已经存在的实现的话,使用method_setImplementation(_:_:)
      一个OC方法仅仅是一个C的函数.该函数携带至少两个参数----self 和 _cmd.举例,给定下列函数
    
      void myMethodIMP(id self, SEL _cmd)
      {
        //implementation ...
        
      }
      
      您可以动态添加该函数到一个类中作为一个方法.(调用 resolveThisMethodDynamically) 像这样:
      
      class_addMethod([self class], @selector(resolveThisMethodDynamically), (IMP) myMethodIMP, "v@:");
      
     **/

    func  class_addMethod(_ cls:AnyClass?,_ name:Selector,_ imp:IMP,_ types: UnsagePointer<Int8>?)
    
    /*
     获取一个类的指定实例方法
     
     aClass: 要检查的类
     aSelector: 要获取方法的选择器
     返回值: 指定方法选择器所对应的方法实现,如果指定类或者它的父类不包含指定选择器的实例方法则返回NULL.
     
     讨论:
     注意这个函数搜索父类的实现,而class_copyMethodList(_:_:)是不会的
     **/

    func class_getInstanceMethod(_ cls:AnyClass?,_ name: Selector) -> Method?
    
    /*
     返回指定类指定名称的类方法
     
     aClass: 一个雷定义的指针.该类包含您想要获取的方法.
     aSelector: 一个SEL类型的指针.该SEL包含你想要获取的方法.
     返回值: Method的指针.对应指定类指定方法选择器的实现.如果该类或者它的父类不包含这个类方法,则返回NULL.
     
     讨论:
     注意这个函数搜索父类的方法实现.class_copyMethodList(_:_:)不搜索
     **/

     func class_getClassMethod(_ cls: AnyClass?, _ name: Selector) -> Method?
     
    /*
     返回一个类的实例方法列表
     
     cls: 想要检测的类
     outCount: 在返回时,包含数组的长度.如果outCounte是NULL,则不返回.
     返回值: Method类型的指针数组.表示该类所有的实例方法实现.
     不包含父类的实例方法实现.该数组包含*outCount指针后面跟一个空终止符.您必须使用free()函数释放该数组.
     
     如果该类没有实例方法.或者该类为Nil,则返回NULL并且*outCount是0
     
     讨论:
     
     获取一个类的类方法列表.使用class_copyMethodList(object_getClass(cls), &count)
     获取父类的方法列表,使用class_getInstanceMethod(_:_:)或者class_getClassMehtod(_:_:)
     **/

    func class_copyMethodList(_ cls: AnyClass?, _ outCount: UnsageMutablePointer<UInt32>?) -> UnsafeMutablePointer<Method>?

    /*
     替换指定类的方法的实现
     
     cls:要修改的类
     name: 一个方法选择器用来确认哪个方法实现要被替换
     imp:新的方法实现.
     types: 一个字符串集合,用来描述参数的类型.必须至少携带两个参数, self 和_cmd, 第二个和第三个字符必须为"@:"(第一个参数为返回类型)
     
     返回值: 修改的类之前的方法实现.
     
     讨论:
    可以通过两种方式实现该函数行为:
    如果方法名不存在,通过call_addMethod(_:_:_:_:)方法的调用添加函数实现.按照types添加参数类型和返回值类型.
    如果方法名存在,他的IMP通过method_setImplementation(_:_:)方法的调用来替换函数实现.types参数被忽略.
    
     **/

    func class_replaceMethod(_ cls:AnyClass?, _ name: Selector,_ imp: IMP, _ types: UnsafePointer<Int8>?) -> IMP?
    
    /*
     获取指定实例方法的实现
    
     cls: 指定类
     name: 方法选择器
     返回值: 返回函数指针,如果cls为Nil,则返回NULL.
     
     讨论:
     
     class_getMethodImplementation(_:_:)可能会比method_getImplementtation(class_getInstanceMethod(cls, name))更快一些.
     
     返回的函数指针可能是运行时的函数，而不是实际的方法实现。例如，如果类的实例不响应选择器，那么返回的函数指针将是运行时消息转发机制的一部分。
     **/ 

    func class_getMethodImplementation(_ cls: AnyClass?,_ name: Selector) -> IMP?
    
    /*
     返回将要被调用的指定消息的函数指针
     
     cls: 指定的类
     name: 一个方法选择器
     返回值: 返回函数指针,如果cls为Nil,则返回NULL.
     **/ 

    func class_getMethodImplementation_stret(_ cls: AnyClass?,_ name: Selector) -> IMP?
    
    /*
    返回一个布尔值表示是否一个实例响应了指定方法
    
    cls : 响应消息的类
    sel : 一个方法选择器
    返回值: 响应方法,则返回true,否则返回false
    
    讨论:
    通常使用NSObject的responds(to:)或者instancesRespond(to:)方法来替换该方法.
     **/
    func class_respondsToSelector(AnyClass?, Selector)

    /*
     给一个类添加一个协议
     
     cls: 要改动的类
     outCount: 要添加到该类的协议
     返回值: 添加成功,返回ture;否则false(比如该类中已经存在该协议)
    
     **/
    func class_addProtocol(_ cls: AnyClass?, _ protocol: Protocol) -> Bool
    
    /*
     给一个类添加一个属性
     
     cls: 要改动的类
     name: 属性名
     attributes: 属性的描述(readonly, nonatomic, asign...)
     attributeCount: 属性描述的数量
     返回值: 添加成功,返回true,否则false(比如已经存在该属性)
     **/

    func class_addProperty(_ cls: AnyClass?, _ name: UnsagePointer<Int8>, _ attributes: UnsafePointer<objc_property_attribute_t>?, _ attributeCounte: UInt32) -> Bool
    
    /*
     替换一个类的属性
     **/
    func class_replaceProperty(_ cls:AnyClass?, _ name: UnsafePointer<Int8>, _ attributes: UnsafePointer<objc_property_attribute_t>?, _ attrubuteCount: UInt32)
    
    /*
     返回一个布尔值 表示是否一个类执行指定协议方法
     
     通常使用NSObject的conforms(to:) 来替换它
     **/

    func class_conformsToProtocol(_ cls: AnyClass?,_ protocol: Protocol?) -> Bool
    /*
     描述一个类包含的协议
     cls: 指定的类
     outCount: 返回数组的长度.如果outCount是NULL,返回值不包含数组的长度.
     返回值: Protocol* 类型的指针数组.描述类中包含的协议.父类的协议不包含其中.数组中包含*outCount指针,并庚随一个空的终止符.必须使用free()函数释放数组.
     
     返回值: 如果cls中没有协议,或者cls为Nil,返回NULL并且*outCount为0
     **/

    func class_copyProtocolList(_ cls: AnyClass?,_ outCount: UnsafeMutablePointer<UInt32>?)  -> AutoreleasingUnsafeMutablePointer<Protocol>?
    
    //返回类定义的版本号
    func class_getVersion(AnyClass?)
    
    //设置一个类定义的版本号
    func class_setVersion(AnyClass, Int32)


```

### 添加Class

```

    /*
     创建一个类和元类
     
     superclass: 新创建类的父类.如果创建一个根类则为Nil
     name: 新创建类的类名.该字符串会被拷贝.
     extraBytes: 类和元类末尾为索引实例变量而分配的字节数.通常为0
     返回值: 新创建的类.或者为Nil如果该类无法被创建(比如使用的名字已经被使用)
     
     讨论:
     可以获取新创建元类的指针通过调用object_getClass(newClass).
     创建一个类.需要先调用objc_allocateClassPair(_:_:_:).然后设置类的特性使用诸如class_addMethod(_:_:_:_:)和class_addIvar(_:_:_:_:_:)这些方法.当你创建完该类后.调用objc_registerClassPair(_:)注册该类后才可以使用.
     
     实例方法和实例变量应该添加到自身类中.类方法应该添加到元类中.
     
     **/

    func objc_allocateClassPair(_ superclass: AnyClass?, _ name: UnsafePointer<Int8>, _ extraBytes: Int) -> AnyClass?
    
    /*
     销毁一个类和它关联的元类.
     
     讨论:
     如果cls类实例或者它的任何子类存在的话,不要调用该函数.
     **/

    func objc_disposeClassPair(_ cls: AnyClass)
    
    //注册一个类 该类已使用objc_allocateClassPair(_:_:_:)方法创建
    func objc_registerClassPair(AnyClass)
    
    /*
     在Foundation框架下的KVO使用
     
     讨论:
     自身不要调用该函数
     **/

    func objc_duplicateClass(AnyClass, UnsafePointer<Int8>, Int)

```

### 与实例一起使用

```

    /*
     读取一个对象中实例变量的值
     讨论:
     如果实例变量名已知的话,object_getIvar(_:_:)比object_getInstanceVariable获取值更快一些.
     **/

    func object_getIvar(_ obj: Any?,_ ivar: Ivar) -> Any?
    
    //设置一个对象实例变量的值
    func object_setIvar(_ obj: Any?,_ ivar: Ivar,_ varlue: Any?)
    
    //返回某对象的类名
    func object_getClassName(_obj: Any?)
    
    //返回实例对象的类对象
    func object_getClass(_ obj: Any?) -> AnyClass?
    
    //设置一个对象的类 返回值为对象之前所属的类对象.
    func object_setClass(_ obj: Any?,_ cls: AnyClass) -> AnyClass?

```

### 获取类定义

```
    //获取已注册类定义的列表
    func objc_getClassList(AutoreleasingUnsafeMutablePointer<AnyClass>?, Int32)
    
    //创建并且返回所有已注册的类定义的指针列表
    func objc_copyClassList(UnsafeMutablePointer<UInt32>?)
    
    //返回某个类的类定义
    func objc_lookUpClass(UnsafePointer<Int8>)
    
    //返回某个类的类定义
    func objc_getClass(UnsafePointer<Int8>)
    
    //返回某个类的类定义
    func objc_getRequiredClass(UnsafePointer<Int8>)
    
    //返回某个类的元类定义
    func objc_getMetaClass(UnsafePointer<Int8>)

```

### 与实例变量一起使用

```
    //返回实例变量名称
    func ivar_getName(Ivar)
    
    //返回实例变量的类型
    func ivar_getTypeEncoding(Ivar)

    //返回一个实例变量相对内存基址的偏移值
    func ivar_getOffset(Ivar)

```

### 关联引用

```
    //使用指定key和关联策略设置某个对象的关联值
    func objc_setAssociateObject(Any, UnsafeRawPointer, Any?, objc_AssociationPoicy)
    
    //返回某个对象该key下对应的关联值
    func objc_getAssociateObject(Any, UnsafeRawPointer)
    
    //移除某个对象所有关联对象
    func objc_removeAssociateObjects(Any)

```

### 与方法一起使用

```
    /*
     返回方法的SEL
     
     讨论:
     获取C字符串的方法名.调用sel_getName(method_getName(method))
     **/

    func method_getName(_ m: Method) -> Selector
    
    //返回IMP类型的函数指针
    func method_getImplementation(_ m: Method) -> IMP
    
    //返回一个字符串描述方法的参数和返回类型
    func method_getTypeEncoding(_ m: Method) -> UnsafePointer<Int8>?
    
    //返回一个字符串描述方法的返回类型
    func method_copyReturnType(Method)
    
    //返回一个字符串描述方法的单个参数类型
    func method_copyArgumentType(Method, UInt32)
    
    //通过引用返回一个字符串描述方法的返回值类型
    func method_getReturnType(Method, UInt32)
    
    //返回一个方法接受的参数数量
    func method_getNumberOfArguments(Method)
    
    //通过引用返回一个字符串描述方法单个参数的类型
    func method_getArgumentType(Method, UInt32, UnsafeMutablePointer<Int8>?, Int)
    
    //返回某个方法的结构描述
    func method_getDescription(Method)
    
    //设置某个方法的实现
    func method_setImplementation(_ m: Method, _ imp: IMP) -> IMP
    
    /*
     交换两个方法的实现
     
     IMP imp1 = method_getImplementation(m1);
     IMP imp2 = method_getImplementation(m2);
     method_setImplementation(m1, imp2);
     method_setImplementation(m2, imp1);
     
     **/ 
    func method_exchangeImplementations(_m1: Method,_ m2: Method)

```

### 与库一起使用

```
    
    //返回所有加载的OC框架和动态库名称  
    func objc_copyImageNames(UnsafeMutablePointer<UInt32>?)
    
    //返回动态库中一个类的原始格式的名称
    func objc_getImageName(AnyClass?)
    
    //返回某个库或者框架中所有类的名称
    func objc_copyClassNameForImage(UnsafePointer<Int8>, UnsafeMutablePointer<UInt32>?)


```

### 与Selectors一起使用

```

    //返回某一方法的名称
    func sel_getName(Selector)
    
    //在OC runtime系统中注册一个方法,映射方法名到一个selector,并且返回selector的值
    func sel_registerName(UnsafePointer<Int8>)
    
    //注册一个方法名到 OC runtime系统中
    func sel_getUid(UnsafePointer<Int8>)
    
    //返回一个布尔值 表示两个方法是否相等
    func sel_isEqual(Selector, Selector)

```

### 与协议一起使用

```
    //返回指定协议
    func objc_getProtocol(UnsafePointer<Int8>)
    
    //以数组形式返回runtime中所有已知协议
    func objc_copyProtocolList(UnsafeMutablePointer<UInt32>)
    
    //创建一个协议实例
    func objc_allocateProtocol(UnsafePointer<Int8>)
    
    //在OC运行时系统中注册一个新协议
    func objc_registerProtocol(Protocol)
    
    //给协议添加一个方法
    func protocol_addMethodDescription(Protocol, Selector, UnsafePointer<Int8>?, Bool, Bool)

    //添加注册过的协议到正在构建的协议中
    func protocol_addProtocol(Protocol, Protocol)
    
    //添加属性到正在构建的协议中
    func protocol_addProperty(Protocol, UnsafePointer<Int8>, UnsafePointer<objc_property_attribute_t>?, UInt32, Bool, Bool)
    
    //返回协议名称
    func protocol_getName(Protocol)
    
    //返回布尔值 表示是否两个协议相等
    func protocol_isEqual(Protocol?, Protocol?)
    
    //返回满足给定协议的方法描述数组
    func protocol_copyMethodDescriptionList(Protocol, Bool, Bool, UnsafeMutablePointer<UInt32>?)
    
    //返回满足给定协议的指定方法的方法描述
    func protocol_getMethodDescription(Protocol, Selector, Bool, Bool)
    
    //返回一个协议声明的属性数组
    func protocol_copyPropertyList(Protocol, UnsafeMutablePointer<UInt32>)
    
    //返回给定协议的指定属性
    func protocol_getProperty(Protocol, UnsafePointer<Int8>, Bool, Bool)
    
    //返回适用某协议的协议数组
    func protocol_copyProtocolList(Protocol, UnsafeMutablePointer<UInt32>)
    
    //返回一个布尔值 表示是否一个协议遵循另一个协议
    func protocol_comformsToProtocol(Protocol?, Protocol?)

```

### 与属性一起使用

```
    //返回属性名
    func property_getName(_ property: objc_property_t) -> UnsafePointer<Int8>
    
    //返回一个属性的的特征字符串
    func property_getAttributes(_ property: objc_property_t) -> UnsafePointer<Int8>
    
    //返回指定特征名的属性值
    func property_copyAttributeValue(_ property: objc_property_t,_ attributeName: UnsafePointer<Int8>) -> UnsafeMutablePointer<Int8>?
    
    //返回指定属性的属性特征数组
    func property_copyAttributeList(_ property: objc_property_t, _ outCount: UnsafeMutablePointer<UInt32>?) -> UnsafeMutablePointer<objc_property_attribute_t>?

```

### OC语言特色的使用

```
    //当在foreach迭代中检测到一个突变时，由编译器插入。
    func objc_enumerationMutation(Any)
    
    //设置当前的突变处理
    func objc_setEnumerationMutationHandler(((Any) -> Void)?)
    
    //给一个函数创建一个指针  当方法被调用的使用调用指定Block
    func imp_implementationWithBlock(Any)
    
    //返回与一个已经创建使用的IMP关联的Block
    func imp_getBlock(IMP)
    
    //取消与已经创建使用的IMP关联的Block
    func imp_removeBlock(IMP)
    
    //加载被弱引用的对象并返回它
    func objc_loadWeak()
    
    //存储一个弱引用变量值
    func objc_storeWeak(AutoreleasingUnsafeMutablePointer<AnyObject?>, Any?)

```

### 类定义数据结构

```
    //在类定义中表示一个方法类型
    typealias Method
    
    //表示一个实例变量类型
    typealias Ivar
    
    //表示一个分类类型
    typealias Category
    
    //表示一个OC声明的属性类型
    typealias objc_property_t
    
    //定义一个OC方法
    struct objc_method_description
    
    //定义一个属性特性
    struct objc_property_attribute_t

```

### 实例数据类型

  这些是表示对象,类,父类的数据类型
    
 -  objc_object 指向一个实例对象的指针
 -  objc_object 表示一个实例对象
 -  objc_super  一个实例对象的父类

```
    //一个类的实例的指针
    struct objc_object
    
    //指定实例对象的父类
    struct objc_super
    
```
