---
title:  YYCategories介绍 
date:  2018-02-09 18:44
categories:
- iOS
tags: 
- YYCategories 
---
## 介绍
功能丰富的 Category 类型工具库

## 分析

###### Code

```
#if __has_include(<YYCategories/YYCategories.h>)
#import <YYCategories/YYCategoriesMacro.h>
#import <YYCategories/NSObject+YYAdd.h>
...
#else
#import "YYCategoriesMacro.h"
#import "NSObject+YYAdd.h"
...
#endif
```
###### 功能

这样做可以提高编译效率.如果指定文件路径,则搜索的更快一些.\
一般我们使用YYCategories导入的时候都是库文件,所以指定目录位置搜索更快一些

```
＃import < > 引用系统文件，它用于对系统自带的头文件的引用，编译器会在系统文件目录下去查找该文件.
#import " "  用户自定义的文件用双引号引用，编译器首先会在用户目录下查找，然后到安装目录中查
```

---

###### Code

```
FOUNDATION_EXPORT double YYCategoriesVersionNumber;
FOUNDATION_EXPORT const unsigned char YYCategoriesVersionString[];
```

###### 功能

例如比较两个字符串

`FOUNDATION_EXPORT` 直接使用`stringInstance == MyFirstConstant`来比较,比较的是指针地址\
`define`使用`[stringInstance isEqualToString:MyFirstConstant]`
比较字符串的每一个字符是否相等\
`FOUNDATION_EXPORT`效率更高

```

NS_ASSUME_NONNULL_BEGIN

/**
 Provides extensions for `UIBarButtonItem`.
 */
@interface UIBarButtonItem (YYAdd)

/**
 The block that invoked when the item is selected. The objects captured by block
 will retained by the ButtonItem.
 
 @discussion This param is conflict with `target` and `action` property.
 Set this will set `target` and `action` property to some internal objects.
 */
@property (nullable, nonatomic, copy) void (^actionBlock)(id);

@end

NS_ASSUME_NONNULL_END

```
该段代码使用了`NS_ASSUME_NONNULL_BEGIN`,`NS_ASSUME_NONNULL_END` 

两个宏中间包含的属性,参数值,返回值,默认是 `nonnull` 类型.

如果想要某个属性,参数值或者返回值为可选类型,则单独在该属性,参数值,或者返回值前单独标明`nullable`.

###### Code

```
#ifdef __cplusplus
#define YY_EXTERN_C_BEGIN  extern "C" {
#define YY_EXTERN_C_END  }
#else
#define YY_EXTERN_C_BEGIN
#define YY_EXTERN_C_END
#endif
```

###### 功能

在C++环境中有定义 `__cplusplus` 这个宏,如果在C++环境中, `YY_EXTERN_C_BEGIN`和 `YY_EXTERN_C_END`中间的代码,编译器用C语言的编译格式来编译.因为 C++ 为了实现函数重载会把函数名和参数等联合起来合成一个中介的函数名，如果 C 函数也被这样编译会出问题.

###### Code
```
#ifndef YY_SWAP // swap two value
#define YY_SWAP(_a_, _b_)  do { __typeof__(_a_) _tmp_ = (_a_); (_a_) = (_b_); (_b_) = _tmp_; } while (0)
#endif
```
###### 功能

交换两个值

###### Code
```
#ifndef YYSYNTH_DUMMY_CLASS
#define YYSYNTH_DUMMY_CLASS(_name_) \
@interface YYSYNTH_DUMMY_CLASS_ ## _name_ : NSObject @end \
@implementation YYSYNTH_DUMMY_CLASS_ ## _name_ @end
#endif
```

###### 功能
如果条件成立,执行断言.

###### Code
```
#ifndef YYSYNTH_DUMMY_CLASS
#define YYSYNTH_DUMMY_CLASS(_name_) \
@interface YYSYNTH_DUMMY_CLASS_ ## _name_ : NSObject @end \
@implementation YYSYNTH_DUMMY_CLASS_ ## _name_ @end
#endif
```

###### 功能

在ios开发过程中，有时候会用到第三方的静态库(.a文件)，OC没有为每个函数（或者方法）定义链接符号，它只为每个类创建链接符号。这样当在一个静态库中使用类别来扩展已有类的时候，链接器不知道如何把类原有的方法和类别中的方法整合起来，就会导致你调用类别中的方法时，会出现selector not recognized的错误，从而导致app闪退。使用这段宏定义他可以虚拟新建一个与名字category 相同.h.m 让编译器 编译通过。即可解决上面的问题。

###### Code
```
#ifndef YYSYNTH_DYNAMIC_PROPERTY_OBJECT
#define YYSYNTH_DYNAMIC_PROPERTY_OBJECT(_getter_, _setter_, _association_, _type_) \
- (void)_setter_ : (_type_)object { \
    [self willChangeValueForKey:@#_getter_]; \
    objc_setAssociatedObject(self, _cmd, object, OBJC_ASSOCIATION_ ## _association_); \
    [self didChangeValueForKey:@#_getter_]; \
} \
- (_type_)_getter_ { \
    return objc_getAssociatedObject(self, @selector(_setter_:)); \
}
#endif
```

###### 功能
使用runtime,合成类别中定义的属性的`Setter` 和 `Getter`方法.

###### Code
```
#ifndef weakify
    #if DEBUG
        #if __has_feature(objc_arc)
        #define weakify(object) autoreleasepool{} __weak __typeof__(object) weak##_##object = object;
        #else
        #define weakify(object) autoreleasepool{} __block __typeof__(object) block##_##object = object;
        #endif
    #else
        #if __has_feature(objc_arc)
        #define weakify(object) try{} @finally{} {} __weak __typeof__(object) weak##_##object = object;
        #else
        #define weakify(object) try{} @finally{} {} __block __typeof__(object) block##_##object = object;
        #endif
    #endif
#endif

#ifndef strongify
    #if DEBUG
        #if __has_feature(objc_arc)
        #define strongify(object) autoreleasepool{} __typeof__(object) object = weak##_##object;
        #else
        #define strongify(object) autoreleasepool{} __typeof__(object) object = block##_##object;
        #endif
    #else
        #if __has_feature(objc_arc)
        #define strongify(object) try{} @finally{} __typeof__(object) object = weak##_##object;
        #else
        #define strongify(object) try{} @finally{} __typeof__(object) object = block##_##object;
        #endif
    #endif
#endif
```

###### 功能
生成一个若引用或者强引用.

Example:
```
    @weakify(self)
    [self doSomething^{
        @strongify(self)
        if (!self) return;
        ...
    }];
```

###### Code
```
static inline void YYBenchmark(void (^block)(void), void (^complete)(double ms)) {
    // <sys/time.h> version
    struct timeval t0, t1;
    gettimeofday(&t0, NULL);
    block();
    gettimeofday(&t1, NULL);
    double ms = (double)(t1.tv_sec - t0.tv_sec) * 1e3 + (double)(t1.tv_usec - t0.tv_usec) * 1e-3;
    complete(ms);
}
```

###### 功能

这个函数还是挺实用的,可以用来计算block内代码的执行时间,通过该函数来测试写的代码执行效率.

返回值为毫秒,double类型.
```
/**
 Profile time cost.
 @param block     code to benchmark
 @param complete  code time cost (millisecond)
 
 Usage:
    YYBenchmark(^{
        // code
    }, ^(double ms) {
        NSLog("time cost: %.2f ms",ms);
    });
 
 */
 ```

###### Code
```
static inline NSDate *_YYCompileTime(const char *data, const char *time) {
    NSString *timeStr = [NSString stringWithFormat:@"%s %s",data,time];
    NSLocale *locale = [[NSLocale alloc] initWithLocaleIdentifier:@"en_US"];
    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
    [formatter setDateFormat:@"MMM dd yyyy HH:mm:ss"];
    [formatter setLocale:locale];
    return [formatter dateFromString:timeStr];
}

/**
 Get compile timestamp.
 @return A new date object set to the compile date and time.
 */
#ifndef YYCompileTime
// use macro to avoid compile warning when use pch file
#define YYCompileTime() _YYCompileTime(__DATE__, __TIME__)
#endif
```

###### 功能

获取编译开始时间.格式为"MMM dd yyyy HH:mm:ss"

###### Code
```
/**
 Submits a block for execution on a main queue and waits until the block completes.
 */
static inline void dispatch_sync_on_main_queue(void (^block)(void)) {
    if (pthread_main_np()) {
        block();
    } else {
        dispatch_sync(dispatch_get_main_queue(), block);
    }
}
```

###### 功能

提交blcok到主队列同步执行.下面提交block到主队列异步执行,使用起来还是蛮方便的.

###### Code (NSArray + YYAdd)
```
/**
 Reverse the index of object in this array.
 Example: Before @[ @1, @2, @3 ], After @[ @3, @2, @1 ].
 */
- (void)reverse;

- (void)reverse {
    NSUInteger count = self.count;
    int mid = floor(count / 2.0);
    for (NSUInteger i = 0; i < mid; i++) {
        [self exchangeObjectAtIndex:i withObjectAtIndex:(count - (i + 1))];
    }
}
```
###### 功能

数组翻转

###### Code (NSDate + YYADD)
```
/**
 Create data from the file in main bundle (similar to [UIImage imageNamed:]).
 
 @param name The file name (in main bundle).
 
 @return A new data create from the file.
 */
+ (nullable NSData *)dataNamed:(NSString *)name;
```

###### 功能

返回main bundle 下指定文件的的二进制数据.

###### Code (NSNumber + YYAdd)
```
/**
 Creates and returns an NSNumber object from a string.
 Valid format: @"12", @"12.345", @" -0xFF", @" .23e99 "...
 
 @param string  The string described an number.
 
 @return an NSNumber when parse succeed, or nil if an error occurs.
 */
+ (nullable NSNumber *)numberWithString:(NSString *)string;
```

###### 功能

将一个字符串对象解析为NSNumber类型对象,很实用.

###### Code (NSObject+YYAdd)
```
+ (BOOL)swizzleInstanceMethod:(SEL)originalSel with:(SEL)newSel {
    Method originalMethod = class_getInstanceMethod(self, originalSel);
    Method newMethod = class_getInstanceMethod(self, newSel);
    if (!originalMethod || !newMethod) return NO;
    
    class_addMethod(self,
                    originalSel,
                    class_getMethodImplementation(self, originalSel),
                    method_getTypeEncoding(originalMethod));
    class_addMethod(self,
                    newSel,
                    class_getMethodImplementation(self, newSel),
                    method_getTypeEncoding(newMethod));
    
    method_exchangeImplementations(class_getInstanceMethod(self, originalSel),
                                   class_getInstanceMethod(self, newSel));
    return YES;
}
```

###### 功能
runtime 黑魔法,方法实现交换.

###### Code
```
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-相关命令"
    //需要操作的代码
#pragma clang diagnostic pop
```

###### 功能

消除编译器警告⚠️

|相关命令      |含义          |
|--------------|--------------|
|"-Wdeprecated-declarations"|弃用的警告⚠️|
|"-Wincompatible-pointer-types"|不兼容指针类型⚠️|
|"-Warc-retain-cycles"|循环引用⚠️|
|"-Wunused-variable"|未使用变量 ⚠️|
|"-Wcovered-switch-default"|未使用default ⚠️|
|...|...|


###### Code (UIBarButtonItem+YYAdd)
```
@interface _YYUIBarButtonItemBlockTarget : NSObject

@property (nonatomic, copy) void (^block)(id sender);

- (id)initWithBlock:(void (^)(id sender))block;
- (void)invoke:(id)sender;

@end

@implementation _YYUIBarButtonItemBlockTarget

- (id)initWithBlock:(void (^)(id sender))block{
    self = [super init];
    if (self) {
        _block = [block copy];
    }
    return self;
}

- (void)invoke:(id)sender {
    if (self.block) self.block(sender);
}

@end


@implementation UIBarButtonItem (YYAdd)

- (void)setActionBlock:(void (^)(id sender))block {
    _YYUIBarButtonItemBlockTarget *target = [[_YYUIBarButtonItemBlockTarget alloc] initWithBlock:block];
    objc_setAssociatedObject(self, &block_key, target, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    
    [self setTarget:target];
    [self setAction:@selector(invoke:)];
}

- (void (^)(id)) actionBlock {
    _YYUIBarButtonItemBlockTarget *target = objc_getAssociatedObject(self, &block_key);
    return target.block;
}
```

###### 功能

通过创建一个中间对象,让UIBarButtonItem的Target指向该对象,同时让self 调用该对象的方法.来达到点击事件调用block而无需再设置selector的目的.

通过这种方式,`YYCategories`将多种类似点击控件,调用selector的方式,修改为block.

> **注意** 使用block给控件添加点击事件后,就不能再给该控件添加selector,否则会引起冲突.

###### Code  (UIColor+YYAdd)
```
#ifndef UIColorHex
#define UIColorHex(_hex_)   [UIColor colorWithHexString:((__bridge NSString *)CFSTR(#_hex_))]
#endif
```
###### 功能

16进制颜色转换
Example: UIColorHex(0xF0F), UIColorHex(66ccff), UIColorHex(#66CCFF88)
支持的格式是如此之多, #RGB #RGBA #RRGGBB #RRGGBBAA 0xRGB ...

###### Code  (UIImage+YYAdd)
```
/**
 Returns a new rotated image (relative to the center).
 
 @param radians   Rotated radians in counterclockwise.⟲
 
 @param fitSize   YES: new image's size is extend to fit all content.
                  NO: image's size will not change, content may be clipped.
 */
- (nullable UIImage *)imageByRotate:(CGFloat)radians fitSize:(BOOL)fitSize;
```

###### 功能

图片翻转,超级实用
...

## 总结

`YYCategories`中对`UIKit`, `Foundation`, `Quartz`中的常用类添加分类,里面还有好多实用的API来供我们项目开发使用,想要进一步了解并使用其中的API,可以参阅`YYCategories`中的头文件.

## 参考资料

[YYCategories](https://github.com/ibireme/YYCategories)
[clang.llvm.org](http://clang.llvm.org/docs/UsersManual.html#diagnostics_pragmas)
