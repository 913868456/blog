
---
title:  Runtime编程指南 
date:  2018-01-16 20:54
categories:
- iOS
tags: 
- Runtime 
---
## 介绍

Objective-C语言从编译、链接、到运行都有很多决策。只要有可能，它都是动态的。这意味着该语言不仅需要编译器，还需要运行时系统来执行编译后的代码。运行时系统作为Objective-C语言的一种操作系统;这就是语言的作用。
本文档关注NSObject类，以及Objective-C程序如何与运行时系统交互。特别是，它检查了在运行时动态加载新类的范例，并将消息转发给其他对象。它还提供关于在程序运行时如何查找对象信息的相关内容。
您应该阅读本文档以了解Objective-C运行时系统如何工作以及如何利用它。但是，通常情况下，您不需要知道和理解这些材料来编写Cocoa应用程序。
## Runtime 版本和平台（省略）

## Runtime交互

OC在三个不同层面跟runtime系统交互: 通过OC源代码;通过Foundation框架下NSObject定义的方法;通过直接调用runtime函数.

### Objective-C 源码

在大多数情况下，运行时系统会自动地在幕后工作。您只需编写和编译Objective-C源代码就可以使用它。

当编译包含Objective-C类和方法的代码时，编译器会创建实现语言动态特性的数据结构和函数调用。数据结构捕获在类和类别定义,协议声明中发现的信息;它们包括在Objective-C编程语言中定义的类对象和协议对象，以及方法选择器、实例变量的模板和从源代码中提取的其他信息。runtime最重要的函数是发送消息的函数，如*消息传递*中所描述的那样。它是由源代码消息表达式调用的。

### NSObject 方法

Cocoa中的大多数对象都是NSObject类的子类，所以大多数对象继承了它定义的方法。(值得注意的例外是NSProxy类;参见*消息转发*以获取更多信息。因此，它的方法建立了每个实例和每个类对象固有的行为。然而，在少数情况下，NSObject类仅仅定义了应该如何做的模板;它没有提供所有必要的代码。

例如，NSObject类定义了一个描述实例方法，该方法返回描述类内容的字符串。这主要用于调试——GDB打印对象命令打印从该方法返回的字符串。NSObject的这个方法的实现不知道类包含什么，所以它返回一个带有对象名称和地址的字符串。NSObject的子类可以实现这个方法来返回更多的细节。例如，Foundation类NSArray返回它所包含的对象的描述列表。

一些NSObject方法简单地查询运行时系统的信息。这些方法允许对象执行内部自检。此类方法的示例是类方法，该类要求对象标识其类;`isKindOfClass:`和`isMemberOfClass:`测试对象在继承层次结构中的位置;`respondsToSelector:`表示一个对象是否可以接受特定的消息;`conformsToProtocol:`表示一个对象是否实现了特定协议中定义的方法;和`methodForSelector:`，它提供了方法实现的地址。像这样的方法给了一个对象自检的能力。

### Runtime 函数

运行时系统是一个动态共享库，其公共接口由位于目录/usr/include/objc中的头文件中的一组函数和数据结构组成。许多这些函数允许您使用plain C来复制编译器在编写Objective-C代码时所做的工作。其他则是通过NSObject类方法导出的功能的基础。这些功能使开发运行时系统的其他接口成为可能，并生成增强开发环境的工具;在Objective-C中编程时不需要它们。然而，在编写Objective-C程序时，一些运行时函数可能会非常有用。所有这些功能都记录在Objective-C运行时引用中。

## 消息传递

本章描述如何将消息表达式转换为`objc_msgSend`函数调用，以及如何通过名称引用方法。然后它解释了如何利用`objc_msgSend`，如果需要的话，可以绕过动态绑定。

### objc_msgSend 函数

在Objective-C中，消息直到运行时才绑定到方法实现。编译器将消息表达式转换为对消息传递函数`objc_msgSend`的调用。该函数接受消息的接收者和消息中提到的方法的名称，即方法selector—作为它的两个主要参数:

```
objc_msgSend(receiver, selector)
```
消息中传送过来的任何参数也通过`objc_msgSend`来处理:

```
objc_msgSend(receiver, selector, arg1, arg2, ...)
```
消息传递函数为实现动态绑定所做的必要事情:

- 它首先查找选择器引用的过程(方法实现)。由于相同的方法可以通过不同的类实现不同的实现，因此它所发现的精确过程取决于接收者的类。
- 然后在调用程序中，将接收对象(指向它的数据的指针)传递，以及为该方法指定的任何参数。
- 最后，它将程序的返回值作为其本身的返回值传递。

> 注意: 编译器通常调用消息传递函数.不要在你写的代码中直接调用它.

消息传递的关键在于编译器为每个类和对象构建的结构。每个类结构包括这两个基本要素:

- 指向父类的指针。
- 一个类分派表。这个表有关联方法选择器和它们识别的方法的类特定地址的条目。`setOrigin`方法的选择器与其实现地址相关联;`display`方法的选择器与其实现地址相关联，等等。

创建新对象时，将分配内存，并初始化其实例变量。对象的首个变量是指向其类结构的指针。这个指针称为isa，它使对象可以访问它的类，并通过该类对它继承的所有类进行访问。

> 虽然这不是语言的一部分，但是需要isa指针来处理Objective-C运行时系统。在结构定义的任何字段中，对象需要“等效”到struct objc_object(在objc/objc.h中定义)。然而，您很少需要创建自己的根对象，而从NSObject或NSProxy继承的对象会自动拥有isa变量。

这些类元素和对象结构在 *Figure 3- 1* 中阐述.

![Figure 3-1 消息传递结构](http://upload-images.jianshu.io/upload_images/3340896-dc0283a1fceff976.gif?imageMogr2/auto-orient/strip)

当消息被发送到一个对象时，消息传递函数会跟随对象的isa指针到类结构，它在分派表中查找方法选择器。如果它在那里找不到选择器，`objc_msgSend`会跟随指向父类的指针，并试图在它的分派表中找到选择器。连续的失败导致`objc_msgSend`爬升类层次结构，直到它到达NSObject类。一旦它定位了选择器，函数调用表中的该方法并将接收对象的数据结构传递给它。

这是在运行时选择方法实现的方式——或者，用面向对象编程的行话来说，方法是动态绑定到消息的。

为了加快消息传递进程，运行时系统将缓存选择器和方法的地址。每个类都有一个单独的缓存，它可以包含用于继承方法的选择器以及类中定义的方法。在搜索调度表之前，消息传递例程首先检查接收对象的类的缓存(关于可能再次使用的方法可能会再次使用的理论)。如果方法选择器在缓存中，消息传递只比函数调用稍微慢一点。一旦程序运行足够长的时间来“预热”它的缓存，它发送的几乎所有消息都会找到一个缓存的方法。当程序运行时，缓存会动态地增长以容纳新的消息。

### 使用隐藏的参数

当`objc_msgSend`找到实现方法的过程时，它调用该过程并将消息中的所有参数传递给它。同时传递隐藏的两个参数:

- 接收对象
- 方法的选择器。
这些参数提供了关于调用它的消息表达式的另一半显式信息。它们被认为是“隐藏的”，因为它们没有在定义方法的源代码中声明。当代码被编译时，它们被插入到实现中。

虽然这些参数没有显式地声明，但是源代码仍然可以引用它们(就像它可以引用接收对象的实例变量一样)。方法将接收对象作为self，并将它的方法选择器作为_cmd。在下面的示例中，_cmd 引用方法选择器作为strange的方法,引用self做为strange消息的接收对象。

```
- strange
{
    id  target = getTheReceiver();
    SEL method = getTheMethod();
 
    if ( target == self || method == _cmd )
        return nil;
    return [target performSelector:method];
}
```
self是两个参数中比较有用的一个。实际上，接收对象的实例变量的方式可以用于方法定义。

### 获取一个方法的地址

规避动态绑定的唯一方法是获取方法的地址，并直接调用它，就像它是一个函数一样。这可能适用于极少数情况下，特定的方法将连续多次执行，并且您希望在每次执行方法时避免消息传递的开销。

使用NSObject类中定义的方法，methodForSelector:，您可以要求一个指向方法实现的指针，然后使用指针来调用该过程。methodForSelector:返回的指针必须小心地转换为合适的函数类型。返回类型和参数类型都应该包含在转换中。

下面的示例展示了实现`setFilled`方法可能调用的流程:

```
void (*setter)(id, SEL, BOOL);
int i;
 
setter = (void (*)(id, SEL, BOOL))[target
    methodForSelector:@selector(setFilled:)];
for ( i = 0 ; i < 1000 ; i++ )
    setter(targetList[i], @selector(setFilled:), YES);
```
传递给流程的前两个参数是接收对象(self)和方法选择器(_cmd)。这些参数隐藏在方法语法中，但是当方法被调用为函数时，必须显式地进行说明。

使用methodForSelector:规避动态绑定可以节省消息传递所需的大部分时间。但是，只有在一个特定的消息重复多次的情况下，才会有显著的节省，如上面所示的for循环。

注意，methodForSelector:由Cocoa运行时系统提供;这不是Objective-C语言本身的特性。

## 动态方法解决方案

本章描述如何动态地提供方法的实现。

### 动态方法解决方案

有些情况下，您可能希望动态地提供方法的实现。例如，Objective-C声明的属性特性(参见Objective-C编程语言中的声明属性)包括@dynamic指令:

```
@dynamic propertyName;
```
它告诉编译器将动态地提供与属性关联的方法。

您可以实现方法resolveInstanceMethod:和resolveClassMethod:为实例和类方法动态地提供给定选择器的实现。

Objective-C方法仅仅是一个C函数，它至少需要两个参数-self和_cmd。您可以使用函数`class_addMethod`将函数添加到类中。因此，给定以下函数:

```
void dynamicMethodIMP(id self, SEL _cmd) {
    // implementation ....
}
```
可以动态地将其添加到一个类作为一个方法(称为resolveThisMethodDynamically)使用resolveInstanceMethod:是这样的:
```
@implementation MyClass
+ (BOOL)resolveInstanceMethod:(SEL)aSEL
{
    if (aSEL == @selector(resolveThisMethodDynamically)) {
          class_addMethod([self class], aSEL, (IMP) dynamicMethodIMP, "v@:");
          return YES;
    }
    return [super resolveInstanceMethod:aSEL];
}
@end
```
转发方法(如消息转发中所描述的)和动态方法解析在很大程度上是正交的。类有机会在转发机制启动之前动态解析方法。如果调用`respondsToSelector`或`instancesRespondToSelector:`方法,动态方法解析器就有机会给选择器提供一个IMP。如果您实现了`resolveInstanceMethod:`但是希望特定的选择器通过转发机制来转发，给这些方法选择器返回NO.

### 动态加载

Objective-C程序可以在运行时加载和链接新的类和类别。新代码合并到程序中，并在开始时对加载的类和类别进行相同的处理。

动态加载可以用来做很多不同的事情。例如，系统首选项应用程序中的各个模块是动态加载的。

在Cocoa环境中，动态加载通常用于允许定制应用程序。其他人可以编写程序在运行时加载的模块——就像接口构建器加载自定义面板和OS X系统首选项应用程序加载自定义的偏好模块一样。可加载模块扩展了应用程序的功能。他们以你允许的方式对它做出贡献，但却无法预见或定义你自己。您提供了框架，但其他人提供了代码。

虽然有一个运行时函数，在Mach-O文件中执行Objective-C模块的动态加载(objc /objc-load.h中定义的objc_loadmodule)，但Cocoa的NSBundle类为动态加载提供了一个更方便的接口，这是面向对象的，并与相关的服务集成在一起。在Foundation框架参考中查看NSBundle类规范，了解关于NSBundle类及其使用的信息。请参阅OS X ABI Mach-O文件格式参考，以获取关于Mach-O文件的信息。

## 消息转发

向不处理该消息的对象发送消息是错误的。然而，在宣布错误之前，运行时系统会再给接收对象一个机会来处理该消息。

### 转发

如果您向一个不处理该消息的对象发送消息，在宣布一个错误之前，运行时将向对象发送对象一个`forwardInvocation:`消息,并携带一个`NSInvocation`对象参数——NSInvocation对象封装了原始消息和传递给它的参数。

您可以实现一个`forwardInvocation:`方法来提供对消息的默认响应，或者以其他方式避免错误。正如其名称所暗示的,`forwardInvocation`通常用于将消息转发到另一个对象。

为了查看转发的范围和意图，请想象以下场景:假设您正在设计一个对象，该对象可以响应一个名为`negotiate`的消息，您希望它的响应包含另一类型对象的响应。通过在您实现`negotiate`方法的主体中传递一个`negotiate`消息到其他对象，您可以轻松完成这一任务。

更进一步，假设您希望您的对象对`negotiate`消息的响应完全在另一个类中实现。实现这一点的一种方法是让您的类继承其他类的方法。然而，这样安排是不可能的。也许有很好的理由来说明为什么不可能.

即使您的类不能继承`negotiate`方法，您仍然可以通过将消息传递给其他类的实例方法来“借用”它:

```
- (id)negotiate
{
    if ( [someOtherObject respondsTo:@selector(negotiate)] )
        return [someOtherObject negotiate];
    return self;
}
```
这样做可能会有点麻烦，特别是如果有许多消息需要您的对象传递给其他对象时。您必须实现一种方法来覆盖您想要从其他类中借用的每个方法。此外，在您编写代码时，您不知道您可能想要转发的完整的消息集是不可能处理的。该集合可能依赖于运行时的事件，并且随着新的方法和类在将来实现，它可能会发生变化。

`forwardInvocation:`提供的第二次机会为这个问题提供了一个的临时解决方案，它是动态的，而不是静态的。它的工作原理是这样的:当一个对象不能响应消息时，因为它没有找到与消息中的选择器匹配的方法，运行时系统通过发送一个`forwardInvocation:`消息来通知对象。每个对象都继承了来自NSObject类的`forwardInvocation:`方法。然而,NSObject版本的方法只是简单地调用`doesNotRecognizeSelector:`。通过重写NSObject的版本并实现您自己的版本，您可以利用`forwardInvocation:`消息将消息转发给其他对象。

要转发一条消息，所有`forwardInvocation:`方法需要做的是:

- 确定消息应该发送到哪里，以及。
- 将原始参数发送到那里.

可以用`invokeWithTarget:`方法发送消息:
```
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    if ([someOtherObject respondsToSelector:
            [anInvocation selector]])
        [anInvocation invokeWithTarget:someOtherObject];
    else
        [super forwardInvocation:anInvocation];
}
```

转发消息的返回值将返回给原始发送方。所有类型的返回值都可以传递给发送方，包括id、结构和双精度浮点数。

`forwardInvocation:`方法可以作为 **未识别** 消息的分发中心，将它们分配给不同的接收者。或者它可以是一个传输站，将所有的消息发送到同一个目的地。它可以将一个消息转换成另一个消息，或者简单地“吞下”一些消息，因此没有响应，没有错误。`forwardInvocation`方法还可以将多个消息合并为一个响应。`forwardInvocation` 是由实现者决定的。它为对象链到转发链提供了一个机会，这也为相关程序设计提供了可能。

> **注意:** `forwardInvocation:`只有在名义接收方不调用现有方法的情况下，该方法才可以处理消息。例如，如果您希望您的对象转发`negotiate`消息给另一个对象，那么它就不能拥有自己的`negotiate`方法。如果是这样，消息将永远不会到达`forwardInvocation:`。

有关转发和调用的更多信息，请参阅基础框架引用中的NSInvocation类规范。

### 转发和多继承

转发模仿继承，并可用于向Objective-C程序提供多继承的一些影响。如图5-1所示，通过转发来响应消息的对象似乎可以借用或“继承”在另一个类中定义的方法实现。
![图5-1](http://upload-images.jianshu.io/upload_images/3340896-e628ceb1736d6121.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在这个例子中，一个战士类的实例将一个协商消息转发给一个外交官类的实例。战士会像外交官一样谈判。它似乎会对谈判的信息作出回应，而且对于所有实际的目的而言，它确实会做出回应(尽管它确实是一名从事这项工作的外交官)。

因此，转发消息的对象从继承层次结构的两个分支中“继承”方法——它自己的分支和响应消息的对象的分支。在上面的例子中，似乎战士层级继承了外交官和它自己的父类。

转发提供了您通常希望从多个继承中获得的大部分特性。然而，两者之间有一个重要的区别:多重继承将不同的功能组合在一个对象中。它倾向于大的、多层面的对象。而转发是将不同的责任分配给不同的对象。它将问题分解成更小的对象，由这些小的对象来处理相关消息。

### Surrogate对象

转发不仅可以模仿多重继承，还可以开发轻量级对象,这个对象可以表示或者涵盖更多实用的功能。Surrogate代表另一个对象，并将消息传递给它。

在Objective-C编程语言中“远程消息传递”中讨论的Surrogate是这样的代理。Surrogate处理消息转发到远程接收器的细节，确保在连接中复制和检索参数值，等等。但它并没有尝试去做其他的事情;它不会复制远程对象的功能，而是简单地给远程对象一个本地地址，一个可以在另一个应用程序中接收消息的地方。

其他类型的Surrogate对象也是可能的。例如，假设您有一个处理大量数据的对象，可能会创建一个复杂的映像或读取磁盘上文件的内容。设置这个对象可能非常耗时，所以您更喜欢在实际需要时或系统资源临时空闲时使用它。与此同时，为了使应用程序中的其他对象能够正常工作，您至少需要一个占位符来支持该对象。

在这种情况下，您可以开始创建，而不是完全的对象，而是一个轻量级的Surrogate。这个对象可以自己做一些事情，比如回答关于数据的问题，但大多数情况下，它只会为较大的对象保留一个位置，当时间到来时，将消息转发给它。当Surrogate的`forwardInvocation:`方法接收一个转发给另一个对象的消息时，它将确保该对象存在，并且如果它不存在，将创建它。对于较大对象的所有消息都通过Surrogate，因此，就程序的其余部分而言，Surrogate和较大的对象将是相同的。

### 转发和继承

尽管转发模仿继承，NSObject类从不混淆两者。方法类似respondsToSelector:和isKindOfClass:只查看继承层次结构，而不关注转发链。例如，如果询问一个战士对象是否响应协商消息，
```
if ( [aWarrior respondsToSelector:@selector(negotiate)] )
    ...
```
答案是否定的，即使它可以在没有错误的情况下接受协商，并且在某种意义上，通过将它们转发给一个外交官来回应。(见图5 - 1)。

在很多情况下，答案是否定的。但事实可能并非如此。如果您使用转发来设置代理对象或扩展类的功能，则转发机制应该像继承一样透明。如果您希望您的对象表现得好像它们确实继承了它们转发消息的对象的行为，那么您将需要重新实现respondsToSelector:和isKindOfClass:方法来包含您的转发算法:

```
- (BOOL)respondsToSelector:(SEL)aSelector
{
    if ( [super respondsToSelector:aSelector] )
        return YES;
    else {
        /* Here, test whether the aSelector message can     *
         * be forwarded to another object and whether that  *
         * object can respond to it. Return YES if it can.  */
    }
    return NO;
}
```
除了`respondsToSelector:` `isMemberOfClass:`和`isKindOfClass:`,`instancesRespondToSelector:`方法也应该实现转发算法。如果使用了协议，那么`conformsToProtocol:`方法也应该被添加到列表中。类似地,如果一个对象将任何远程转发消息接收,它应该有一个版本的`methodSignatureForSelector:`可以返回准确的描述方法,最终回复转发消息;例如,如果一个对象将消息转发给Surrogate,您将实现`methodSignatureForSelector:`如下:
```
- (NSMethodSignature*)methodSignatureForSelector:(SEL)selector
{
    NSMethodSignature* signature = [super methodSignatureForSelector:selector];
    if (!signature) {
       signature = [surrogate methodSignatureForSelector:selector];
    }
    return signature;
}
```
您可能会考虑将转发算法放在私有代码的某个地方，并拥有所有这些方法，`forwardInvocation:`包括，调用它。
> 注意: 这是一种先进的技术，只适用于没有其他解决方案的情况下。它不是用来代替继承的。如果您必须使用这种技术，请确保您完全理解了正在转发的类的行为和转发的类。
本节中提到的方法在Foundation框架引用中的NSObject类规范中进行了描述。有关`invokeWithTarget`的信息,在Foundation框架引用中查看NSInvocation类规范。

## 类型编码

为了帮助运行时系统，编译器为字符串中的每个方法编码返回和参数类型，并将字符串与方法选择器关联起来。它所使用的编码方案在其他上下文中也很有用，因此可以通过公开的@encode()编译器指令。当给定一个类型规范时，@encode()将返回一个编码该类型的字符串。类型可以是基本类型，例如int、指针、标记的结构体或union，或者类名称——任何类型，实际上都可以用作对C sizeof()操作符的参数。

```
char *buf1 = @encode(int **);
char *buf2 = @encode(struct key);
char *buf3 = @encode(Rectangle);
```
下表列出了类型代码。请注意，它们中的许多都与为存档或分发目的而编码对象时使用的代码重叠。但是，这里列出的代码是在编写代码时不能使用的，而且在编写代码时，您可能想要使用一些代码，这些代码不是由@encode()生成的。(请参阅Foundation框架参考中的NSCoder类规范，以获得关于用于存档或分发的编码对象的更多信息)。

**Table 6-1** Objectiv-C 类型编码

|Code|含义|
|-----|----|
|c| char|
|i| int|
|s| short|
|l|long|
|q|long long |
|C|unsingned char|
|I|unsigned int|
|S|unsigned short|
|L|unsigned long|
|Q|unsigned long long|
|f|float|
|d|double|
|B|C++ bool 或者 C99 _Bool|
|v|void|
|*|A character string(char *)|
|@|object(不管是静态类型或者id类型)
|#|class object (Class)|
|:|method selector(SEL)|
|[array type]|array|
|{name=type...}|structure|
|(name=type...)|union|
|bnum|A bit field of *num* bits|
|^type|A pointer to type|
|?|unknown type(among other things, this code is used for function pointers)|

>**重要**: Objective-C 不支持 long double 类型.@encode(long double)返回 d,编码形式跟 double一样.

数组的类型代码括在方括号内;数组中元素的数量是在数组类型之前，在打开括号之后立即指定的。例如，浮点数的12个指针的数组将被编码为:

```
[12^f]
```
结构是在大括号内指定的，而在括号内的结合。结构标记首先列出，然后是一个等号和序列中列出的结构域的代码。例如,结构体
```
typedef struct example {
    id   anObject;
    char *aString;
    int  anInt;
} Example;
```
会像这样编码:
```
{example=@*i}
```
同样的编码结果，无论定义的类型名称(Example)还是结构标记(example)都传递给@encode()。结构指针的编码包含与结构字段相同的信息量:
```
^{example=@*i}
```
然而,另一层间接移除了内部具体的类型:
```
^^{example}
```
对象被看做结构体.举例来说,传递一个NSObject类名可以这样编码:
```
{NSObject=#}
```
NSObject类只声明了一个类的实例变量isa。

注意，虽然@encode()指令没有返回它们，但是运行时系统使用表6-2中列出的附加编码，用于在协议中声明方法的类型限定符。

**Table 6-2** Objective-C 方法编码

|Code|含义|
|---|---|
|r|const|
|n|in|
|N|inout|
|o|out|
|O|bycopy|
|R|byref|
|V|oneway|


## 属性声明

当编译器遇到属性声明(在Objective-C编程语言中看到已声明的属性)时，它会生成与封装类、类别或协议相关的描述性元数据。您可以使用支持在类或协议上查找属性的函数来访问此元数据，获取属性的类型为@encode字符串，并将属性的属性列表复制为C字符串数组。每个类和协议都有一个声明的属性列表。
### 属性 类型和函数
属性结构为属性描述符定义了一个不透明的句柄。
```
typedef struct objc_property *Property;
```
您可以使用函数`class_copyPropertyList`和`protocol _copypropertylist`检索与类(包括已经加载的类别)关联属性的数组，以及一个协议:
```
objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount)
objc_property_t *protocol_copyPropertyList(Protocol *proto, unsigned int *outCount)
```
例如，给定以下类声明:
```
@interface Lender : NSObject {
    float alone;
}
@property float alone;
@end
```
您可以使用以下方法获取属性列表:
```
id LenderClass = objc_getClass("Lender");
unsigned int outCount;
objc_property_t *properties = class_copyPropertyList(LenderClass, &outCount);
```
您可以使用`property_getName`函数来发现属性的名称:
```
const char *property_getName(objc_property_t property)
```
您可以使用函数`class_getProperty`和`protocol _getproperty`来获得类和协议中给定名称的属性的引用:
```
objc_property_t class_getProperty(Class cls, const char *name)
objc_property_t protocol_getProperty(Protocol *proto, const char *name, BOOL isRequiredProperty, BOOL isInstanceProperty)
```
您可以使用`property_getAttributes`函数来发现属性的名称和@encode类型字符串。有关编码类型字符串的详细信息，请参阅类型编码;有关此字符串的详细信息，请参见属性类型字符串和属性属性描述示例。
```
const char *property_getAttributes(objc_property_t property)
```
将这些组合在一起，您可以使用以下代码打印与类关联的所有属性:
```
id LenderClass = objc_getClass("Lender");
unsigned int outCount, i;
objc_property_t *properties = class_copyPropertyList(LenderClass, &outCount);
for (i = 0; i < outCount; i++) {
    objc_property_t property = properties[i];
    fprintf(stdout, "%s %s\n", property_getName(property), property_getAttributes(property));
}

```

### 属性 类型字符串
您可以使用`property_getAttributes`函数来发现属性的名称、@encode类型字符串和属性的其他属性。

字符串以一个T开头，后面是@encode类型和一个逗号，最后是一个V，后面跟着一个支持实例变量的名称。在他们之间，填充属性描述符，由逗号分隔:

详细内容请看[**Table 7-1**](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html)和属性特性描述示例

## 总结
本文主要关注关注runtime 消息的动态解析与转发，主要包含三个方法：
- `resolveInstanceMethod:` 
当实例对象无法找到 sel 实现时，首先调用此方法，对sel做处理
- `resolveClassMethod:` 
当类对象无法找到 sel 实现时，首先调用此方法，对sel做处理
-  `forwardTargetForSelector:`
sel 仍未处理，接着调用此方法，在这里可以对sel做处理
-  `methodSignatureForSelector:`
sel 仍未处理，runtime会通过`methodSigntureForSelector`方法尝试获取本次消息调用的具体环境信息，包括消息的参数与返回值类型。并封装成NSInvocation对象。我们可以在`forwardInvocation`方法内部对该对象作进一步的处理，并使之能够成功的完成消息处理。如果末能成功获取NSInvocation对象，那么程序就会发送`doesNotRecognizeSelector`消息抛出`unrecognized Selector send to xxx`的异常。


关于runtime的更多有趣的使用，可以在[Objective-C Runtime](https://www.jianshu.com/p/6a72f21bf521)中寻找对应的API来探索，里面有runtime 各方法的说明。

## 代码
[Runtime_Demo](https://github.com/913868456/OCDemo)

## 参考资料
[iOS runtime之消息机制](https://www.jianshu.com/p/e6a9492995a5)
[forwardTargetForSelector:](https://developer.apple.com/documentation/objectivec/nsobject/1418855-forwardingtargetforselector)
[Objective-C Runtime Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048-CH1-SW1)
