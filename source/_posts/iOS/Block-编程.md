
---
title:  Block编程
date: 2017-12-19 16:39
categories:
- iOS
tags: 
- Block
---

## 介绍

Block对象是一个C语言层面的语法和运行时特色.Block类似于标准的C函数,但是除了包含可执行代码之外，它们还可以包含变量绑定到自动(栈)或托管(堆)内存。因此，Block可以维护一组状态(数据)，以便在执行时影响行为。

可以使用block组成函数表达式.这些表达式可以传递给API,可选地存储,并且通过多线程使用.Block作为一个回调非常有用.因为Block不仅可以包含在回调上执行的代码,还可以包含执行过程中需要用到的数据.

在OS X v10.6 和 iOS 4.0 之后可以使用block. block runtime 是 开源的,并且在 [LLVM's compiler-rt subproject respository](http://llvm.org/svn/llvm-project/compiler-rt/trunk/)中可以找到
Block同样被表示为标准的C语言工作组.由于Objective-C 和 C ++ 都是派生自C. Block在这三种语言中都能使用.

通过阅读这个文档去学习什么是block对象,如何在C,C++,OC中使用.

这篇文档的组成

这个文档包含以下章节:

- [开始使用Block]() 提供关于Block的介绍一个快速.实用的使用.
- [概念通览]() 提供关于Block概念方面的介绍
- [声明和创建Block]() 展示如何声明block变量和如何实现block
- [Block 和 变量]() 描述了Block和变量之间的交互,还有定义__block存储类型的修改者.
- [使用Block]() 阐述了Block的多种使用方式


# 开始使用Block

下面内容帮助您使用实用的例子开始使用Block.

## 声明和使用一个Block

使用^操作符去声明一个block变量并且去表示一个block字面量的开始.Block内容包含在{}中.正如这个例子中展示的(跟在C语言中一样, ;表示声明的结束):

```
int multiplier = 7;

int (^myBlock)(int)  = ^(int num){
    
    return num * multiplier;
};

```

示例可以用下图来阐述:

![示例](http://upload-images.jianshu.io/upload_images/3340896-2b3f81f0fddab136.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意block能够在它定义域内作为变量使用.如果你声明一个block变量,你可以像使用函数一样使用它.

```
int multiplier = 7;

int (^myBlock)(int) = ^(int num) {
    
  return num * multiplier;  
};

printf("%d", myBlock(3));

// prints "21"
```

## 直接使用一个Block

在许多情况下,您不需要声明block变量;只需要在需要作为参数的地方简单写一个block字面量即可.下面的例子使用 qsort_b 函数.qsort_b 函数类似于标准的 qsort_r 函数,但是携带一个block作为最后的参数.

```
char *myCharacters[3] = {"TomJohn", "George", "Charles codomine"};

qsort_b(myCharacters, 3, sizeof(char *), ^(const void *l, const void *r){
    
   char *left = * (char **)l;
   char *right = *(cahr **)r;
   
   return strncmp(left, right, l);
    
});

// myCharacters is now {"Charles Condomine", "George", "TomJohn"}
```

## Blocks with Cocoa

在Cocoa框架下的一些方法会携带一个block作为参数,通常用来在一个集合内对象执行操作,或者作为一个操作结束后的回调来使用.下面的例子展示了如何在NSArray的`sortedArrayUsingComparator:`方法中使用一个Block.该方法携带单个参数-Block.为了方面阐述.在这个例子中,block被定义为一个NSComparator类型的本地变量:

```
NSArray *stringsArray = @[@"string 1", @"String 21", @"string 12", @"String 11", @"String 02"];

static NSStringCompareOptions comparisonOptions = NSCaseInsensitiveSearch | NSNumericSearch | NSWidthInsensitiveSearch | NSForceOrderingSearch;

NSLocale *currentLocale = [NSLocale currentLocale];

NSComparator finderSortBlock = ^(id string1, id stirng2) {
    
    NSRange string1Range = NSMakeRange(0, [string1 length]);
    return [stirng1 compare:string2 options:comparisonOptions range:string1Range locale:currentLocale];
};

NSArray *finderSortArray = [stringsArray sortedArrayUsingComparator:finderSortBlock];

NSLog(@"finderSortArray:%@", finderSortArray);

/*
 
 output:
 
 finderSortArray: (
 
    "string 1",
    "String 02",
    "String 11",
    "string 12",
    "String 21"
 )
 */

```

## __block 变量

block的一个强大的特性是在同一作用域内,它们能够修改变量.使用__block存储类型修饰符标记一个在block中可以修改的外部变量. 改变 **Blocks with Cocoa**中的示例,可以使用一个block变量去计算多少个字符串被用来比较. 为了阐述方便,在这个例子中直接使用block,使用**currentLocale**作为block中的一个只读变量.

```
NSArray *stringsArray = @[ @"string 1",
                          @"String 21", // <-
                          @"string 12",
                          @"String 11",
                          @"Strîng 21", // <-
                          @"Striñg 21", // <-
                          @"String 02" ];
 
NSLocale *currentLocale = [NSLocale currentLocale];
__block NSUInteger orderedSameCount = 0;
 
NSArray *diacriticInsensitiveSortArray = [stringsArray sortedArrayUsingComparator:^(id string1, id string2) {
 
    NSRange string1Range = NSMakeRange(0, [string1 length]);
    NSComparisonResult comparisonResult = [string1 compare:string2 options:NSDiacriticInsensitiveSearch range:string1Range locale:currentLocale];
 
    if (comparisonResult == NSOrderedSame) {
        orderedSameCount++;
    }
    return comparisonResult;
}];
 
NSLog(@"diacriticInsensitiveSortArray: %@", diacriticInsensitiveSortArray);
NSLog(@"orderedSameCount: %d", orderedSameCount);
 
/*
Output:
 
diacriticInsensitiveSortArray: (
    "String 02",
    "string 1",
    "String 11",
    "string 12",
    "String 21",
    "Str\U00eeng 21",
    "Stri\U00f1g 21"
)
orderedSameCount: 2
*/

```
详细内容请看 **Blocks 和 变量** 这一章节

# 概念通览

Block对象为您提供一种方式去创建一个ad hoc函数体作为在C,和C的衍生语言中的表达方式.在其他的语言和环境中.一个block对象有时也叫做闭包.

## Block功能

block是一个内联的匿名代码集合:

- 像函数一样有一个类型参数列表
- 拥有推导的或者声明的返回类型
- 能够捕获定义作用域内的状态
- 能够可选地修改作用域内的状态
- 能够与在同一作用域内定义的其他Block共享修改的可能性
- 能够在(栈结构)已经被销毁后,仍然可以继续分享和修改(栈结构)下定义的状态.

您可以拷贝一个block甚至传递它到其他线程用来延迟执行.在生命周期内编译器和运行时将所有被引用的变量保存副本。

## 使用

Block 表示一个小的,内部包含的代码片段.正因为这样,他们常被用来作为执行并行工作单元的一种方式.或者作为集合中的项目,或者作为一个操作结束后的回调.

Blocks作为传统的回调函数主要有两个原因:

1. 它们允许您在方法实现的上下文中执行编写的代码.
   Block通常作为框架方法的参数
2. 它们允许作为方法的局部变量.
   与其使用需要一个包含所有需要执行操作的上下文信息的数据结构的回调函数，您只需直接使用该局部变量.

# 声明和创建 Blocks

## 声明一个Block 引用

Block变量持有Block的引用.使用声明一个函数指针类似的语法声明它们.除了使用^代替*.block类型和C类型系统兼容.下面的block变量声明都是有效的:

```
void (^blockReturningVoidWithVoidArgument)(void);
int (^blockReturningIntWithIntAndCharArguments)(int, char);
void (^arrayOfTenBlocksReturningVoidWithIntArgument[10])(int);

```
Block同样支持可变参数.一个block如果不含参数必须在参数列表中指明void.

在许多地方,使用一个类型别名声明一个Block类型能够更好地使用block.

```
typedef float (MyBlockType)(float, float);

MyBlockType myFirstBlock = // ...;
MyBlockType mySecondBlock = // ...;

```

## 创建一个Block

使用^操作符表示一个block字面量的开始.它后面通常使用()来包含一个参数列表.block体包含在{}中.下面的例子定义了一个简单的block,block末尾用;结尾.

```
float (^oneFrom)(float);

oneFrom = ^(float aFloat) {
  
  float result = aFloat - 1.0;
  return result;
    
};

```

如果没有明确声明一个Block表达式的返回值类型,它会自动推导其类型.如果返回值是被推导的并且参数列表是void,您也可以忽略void参数.当出现多个返回语句时，它们必须完全匹配(在必要时使用转换)。

## 全局Block

在文件层面,您可以使用一个block作为全局字面量

```

#import <stdio.h>

int GlobalInt = 0;

int (^getGlobalInt)(void) = ^(return GlobalInt;);

```

# Blocks 和 变量

本节主要描述blocks和变量之间的交互,包括内存管理.

## 变量类型

在block体的代码内,变量可以用5种不同方式处理.
您可以参考三种标准的变量类型,就像从函数中得到的那样.

- 全局变量,包含静态局部变量
- 全局函数(技术上来讲不是变量)
- 来自作用域内的局部变量和参数

Blocks同样支持两种其他类型的变量:

1.在函数层面是__block 变量.__block变量在block内是可变的,如果正在引用的block被拷贝到堆上,__block变量会被保存起来.
2.**const** 导入的变量.

最后,在一个方法实现内,block可能会引用OC实例变量---请看下面的 [对象和Block变量]() 章节.

下面的规则适用于block中使用的变量:

1. 全局变量是可访问的,包含作用域内存在的静态变量.
2. 传递到block中的参数是可访问的(就像函数的参数一样)
3. 作用域内的栈变量(非静态变量)被捕获作为常量.       (block内使用的外部栈存储的变量(非静态变量)在block内部会作为常量来使用)
变量的值在程序中block表达式的位置上捕获的.在嵌套block中,该值从最近的作用域内被捕获.
4. __block修饰的局部变量是通过引用来提供访问的,并且这些变量在block内部是可修改的.
任何作用域内的改变都会被映射,包括在同一作用域内其他定义的block的改变同样也会被映射.这些内容在"__block 存储类型"中被详细讨论.
5. block作用域内声明的局部变量,其行为跟函数内的局部变量几乎相同.
block的每次调用都会提供那个变量的新的副本. 这些变量依次作为在block内的常量或者引用变量来使用.

下面的例子阐述了非静态局部变量的使用:

```
int x = 123;

vooid (^printXAndY)(int) = ^(int y) {

  printf("%d %d\n", x,y);  
};

printXAndY(456); //prints: 123 456
```
如上所述, 试着在blocke内给x赋值会造成错误:

```
int x = 123;

void (^printXAndY)(int) = ^(int y) {
    
    x = x + y; // error
    printf("%d %d", x, y);  
};

```

想要在block内部对外部变量进行修改,您可以使用__block存储类型修饰符.

## __block 存储类型

您可以使用__block 修饰符使block外部导入的变量在其内部可修改.__block存储类似于局部变量的寄存器,内部的自动,静态存储类型。
存储区域内的__block变量会与作用域内的变量、所有的block、block副本之间共享.因此,在末尾声明的block副本存在,那么在栈结构销毁时,这块内存将会幸存.同一作用域内的多个block可以同时使用一个共享变量.
作为优化.block存储从栈开始,就像block本身一样.如果使用Block_copy复制,则会将变量复制到堆中.因此,__block变量的地址可能随时变化.
对于__block变量有两个进一步的限制:它们不能是可变数组，也不能是包含C99可变长度数组的结构体。

下面的例子阐述了一个__block变量的使用:

```
__block int x = 123; // x lives in block storage

void (^printXAndY)(int) = ^(int y) {
    
    x = x + y;
    printf("%d %D\n", x, y);
};

printXAndY(456); // prints: 579 456

// x is now 579

```

下面的例子展示了block和几种类型变量的交互:

```

extern NSInteger CounterGlobal;
static NSInteger CounterStatic;

{
    NSInteger localCounter = 42;
    __block char localCharacter;
    
    void (^aBlock)(void) = ^(void) {
      ++CounterGlobal;  
      ++CounterStatic;
      CounterGlobal = localCounter; // localCounter fixed at block creation
      localCharacter = 'a'; // sets localCharacter in enclosing scope    
    };
    
    ++localCounter; //unseen by the block
    localCharacter = 'b';
    
    aBlock();  // execute the block 
    // localCharacter now 'a'
}

```

## 对象和Block变量

### Objective-C 对象

当一个block被复制,它会对block内的对象变量创建强引用.如果在方法实现内使用一个block:
- 如果您通过引用访问一个实例变量,会创建一个强引用指向self;
- 如果您通过值访问一个对象变量,会创建一个强引用指向该变量.

下面的例子阐述了两个不同的情况:

```
dispathch_async(queue, ^{
   
   // instanceVariable is userd by reference, a strong reference is made to self 
   
   doSomethingWithObject(instanceVariable);
    
});

id localVariable = instanceVariable;

dispatch_async(queue, ^{
   /*
   
    localVariable is used by value, a strong reference is made to localvariable (and not to self)
   
   */ 
   doSomethingWithObject(localVariable);
    
});

```
如果想要在block内修改外部的局部对象变量,您可以使用__block标记该变量.

### C++对象

一般来说，可以在一个block中使用c++对象。在成员函数中，对成员变量和函数的引用通过隐式导入该指针，因此看起来是可变的。如果一个block被复制，有两个考虑因素:

如果您有一个__block存储类，它是一个基于堆栈的c++对象，那么通常使用的是copy构造函数。
如果您在一个block中使用任何其他c++基于堆栈的对象，那么它必须有一个const copy构造函数。然后使用该构造函数复制c++对象。

### Blocks

当您拷贝一个block时,在block内的其他block引用都会被拷贝如果必要的话.如果您在block内拥有一个block变量或者引用了一个block,那个block将会被拷贝.

# 使用Blocks

## 调用一个Block

声明一个block变量,将其作为一个函数,如下所示:

```

int (^oneFrom)(int) = ^(int anInt){
    
    return anInt - 1;
};

printf("1 from 10 is %d", oneFrom(10));

// Prints "1 from 10 is 9"

float (^distanceTraveled)(float, float, float) = ^(float startingSpeed, float acceleration, float time) {
    
    float distance = (startingSpeed * time) + (0.5 * acceleration * time * time);
    return distance;
};

float howFar = distanceTraveled(0.0, 9.8, 1.0);
// howFar = 4.9
```
## 使用Block作为函数参数

```
char *myCharacters[3] = { "TomJohn", "George", "Charles Condomine" };
 
qsort_b(myCharacters, 3, sizeof(char *), ^(const void *l, const void *r) {
    char *left = *(char **)l;
    char *right = *(char **)r;
    return strncmp(left, right, 1);
});
// Block implementation ends at "}"
 
// myCharacters is now { "Charles Condomine", "George", "TomJohn" }
```
注意block包含在函数的参数列表中.

## 使用Block作为方法参数

Cocoa 提供了许多使用block的方法. 您将一个block作为方法参数使用就像使用其他参数一样. 

```
NSArray *array = @[@"A", @"B", @"C", @"A", @"B", @"Z", @"G", @"are", @"Q"];
NSSet *filterSet = [NSSet setWithObjects: @"A", @"Z", @"Q", nil];
 
BOOL (^test)(id obj, NSUInteger idx, BOOL *stop);
 
test = ^(id obj, NSUInteger idx, BOOL *stop) {
 
    if (idx < 5) {
        if ([filterSet containsObject: obj]) {
            return YES;
        }
    }
    return NO;
};
 
NSIndexSet *indexes = [array indexesOfObjectsPassingTest:test];
 
NSLog(@"indexes: %@", indexes);
 
/*
Output:
indexes: <NSIndexSet: 0x10236f0>[number of indexes: 2 (in 2 ranges), indexes: (0 3)]
*/

```

```
__block BOOL found = NO;
NSSet *aSet = [NSSet setWithObjects: @"Alpha", @"Beta", @"Gamma", @"X", nil];
NSString *string = @"gamma";
 
[aSet enumerateObjectsUsingBlock:^(id obj, BOOL *stop) {
    if ([obj localizedCaseInsensitiveCompare:string] == NSOrderedSame) {
        *stop = YES;
        found = YES;
    }
}];
 
// At this point, found == YES

```

## 拷贝Blocks

通常下,不需要copy或者retain一个block.当想要在block销毁后仍使用block时，才需要copy这个block.拷贝会将一个block移到堆中.

可以使用c函数拷贝和释放block"

```

Block_copy();
Block_release();

```
为避免内存泄露,Block_copy()和Block_release()必须成对出现.

## 格式注意

block字面量是通过一个局部栈数据结构的地址来表示一个block,因此局部栈数据结构的作用域仅限于大括号内。所以应避免以下方式的使用：
```
void dontDoThis() {
    void (^blockArray[3])(void);  // an array of 3 block references
 
    for (int i = 0; i < 3; ++i) {
        blockArray[i] = ^{ printf("hello, %d\n", i); };
        // WRONG: The block literal scope is the "for" loop.
    }
}

void dontDoThisEither() {
    void (^block)(void);
 
    int i = random():
    if (i > 1000) {
        block = ^{ printf("got i at: %d\n", i); };
        // WRONG: The block literal scope is the "then" clause.
    }
    //...
}
```

## 调试

您可以设置断点来执行block中的单步操作.也可以在一个GDB会话中使用invoke-block调用一个block.如下所述:

```

$ invoke-block myBlock 10 20

```
如果您想要传一个c字符串,必须引用它.举例来说,传递一个 this string 给 doSomethingWithString block,按下面方式去写:

```
$ invoke-block doSomethingWithString "\"this string \""
```

# 总结
在Objective-C中,使用block,注意以下几点:

- 使用类型别名来定义一个Block类型,使用起来更方便.
- __block修饰的外部变量,在block内部可以被修改.（修饰的变量不能是可变数组，也不能是包含C99可变长度数组的结构体）
- 每次调用block,都会拷贝该block一次,如果嵌套block,那么内部的block也会被拷贝.
- block进行拷贝操作时,会给内部对象创建一个强引用.如果该对象是实例变量,则创建self的强引用,如果是对象变量,则创建该对象变量的强引用.

# 参考资料
[Blocks Programming Topics](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502-CH1-SW1)



