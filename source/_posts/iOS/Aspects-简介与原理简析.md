
---
title:  AOP简介及Aspects介绍
date:  2018-03-29 00:01
categories:
- iOS
tags: 
- 切面编程 
---

## AOP简介

![aop](https://upload-images.jianshu.io/upload_images/3340896-1096322e544ec637.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在软件业，AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

## Aspects 简介
![aspects](https://upload-images.jianshu.io/upload_images/3340896-cf3f479419055275.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一个轻量的单一的AOP库, 它让我们能够使用 method swizzling 技术 为每个类或者实例对象的方法添加执行代码.

### Aspects 在 iOS 开发常用使用场景

- 监测用户点击事件
- 记录用户某一页面的留存
- 用户使用频率
...

### 示例代码
Aspects 能够在调试的时候被用来动态添加日志
```
[UIViewController aspect_hookSelector:@selector(viewWillAppear:) withOptions:AspectPositionAfter usingBlock:^(id<AspectInfo> aspectInfo, BOOL animated) {
    NSLog(@"View Controller %@ will appear animated: %tu", aspectInfo.instance, animated);
} error:NULL];
```

## 原理简析

讲解原理前我们先熟悉一下`Objective-C`的消息转发流程
### 消息转发流程:
- `resolveInstanceMethod:` 
当实例对象无法找到 sel 实现时，首先调用此方法，对sel做处理
- `resolveClassMethod:` 
当类对象无法找到 sel 实现时，首先调用此方法，对sel做处理
-  `forwardTargetForSelector:`
sel 仍未处理，接着调用此方法，在这里可以对sel做处理
-  `methodSignatureForSelector:`
sel 仍未处理，runtime会通过`methodSigntureForSelector`方法尝试获取本次消息调用的具体环境信息，包括消息的参数与返回值类型。并封装成NSInvocation对象。我们可以在`forwardInvocation`方法内部对该对象作进一步的处理，并使之能够成功的完成消息处理。如果未能成功获取`NSInvocation`对象，那么程序就会发送`doesNotRecognizeSelector`消息抛出`unrecognized Selector send to xxx`的异常

### 原理
> **重要** 
Aspects 利用 `Objective-C` 的消息转发机制,在对象或类调用`selector`的时候,直接将其`IMP`替换为`objc_msgForward`,从而走消息转发流程,通过hook `forwardInvocation:`方法,将其`IMP`替换为自定义的`IMP`从而实现执行代码的添加和方法的替换.

### 核心代码

- 替换selector的IMP,执行 objc_msgForward 函数,直接走消息转发
```
 if (!aspect_isMsgForwardIMP(targetMethodIMP)) {
        ...
        ...
        class_replaceMethod(klass, selector, aspect_getMsgForwardIMP(self, selector), typeEncoding);
        AspectLog(@"Aspects: Installed hook for -[%@ %@].", klass, NSStringFromSelector(selector));
    }
```

- hook `forwardInvocation:`方法,将其IMP替换为`__ASPECTS_ARE_BEING_CALLED__`

```
static NSString *const AspectsForwardInvocationSelectorName = @"__aspects_forwardInvocation:";
static void aspect_swizzleForwardInvocation(Class klass) {
    NSCParameterAssert(klass);
    // If there is no method, replace will act like class_addMethod.
    IMP originalImplementation = class_replaceMethod(klass, @selector(forwardInvocation:), (IMP)__ASPECTS_ARE_BEING_CALLED__, "v@:@");
    if (originalImplementation) {
        class_addMethod(klass, NSSelectorFromString(AspectsForwardInvocationSelectorName), originalImplementation, "v@:@");
    }
    AspectLog(@"Aspects: %@ is now aspect aware.", NSStringFromClass(klass));
}

```
- `forwardInvocation:`方法执行IMP`__ASPECTS_ARE_BEING_CALLED__`

```
// This is the swizzled forwardInvocation: method.
static void __ASPECTS_ARE_BEING_CALLED__(__unsafe_unretained NSObject *self, SEL selector, NSInvocation *invocation) {
    
    ...
    ...

    // Before hooks.
    aspect_invoke(classContainer.beforeAspects, info);
    aspect_invoke(objectContainer.beforeAspects, info);

    // Instead hooks.
    BOOL respondsToAlias = YES;
    if (objectContainer.insteadAspects.count || classContainer.insteadAspects.count) {
        aspect_invoke(classContainer.insteadAspects, info);
        aspect_invoke(objectContainer.insteadAspects, info);
    }else {
        Class klass = object_getClass(invocation.target);
        do {
            if ((respondsToAlias = [klass instancesRespondToSelector:aliasSelector])) {
                [invocation invoke];
                break;
            }
        }while (!respondsToAlias && (klass = class_getSuperclass(klass)));
    }

    // After hooks.
    aspect_invoke(classContainer.afterAspects, info);
    aspect_invoke(objectContainer.afterAspects, info);

    ...
    ...
}
```

- 在上一步中会根据`AspectOptions`按顺序执行block内代码，内部`forwardInvocation`方法调用时获取`NSInvocation`信息，然后用该信息生成一个参数相同的`blockInvocation`，再调用`invokeWithTarget`方法，来执行block内代码

```
- (BOOL)invokeWithInfo:(id<AspectInfo>)info {
    NSInvocation *blockInvocation = [NSInvocation invocationWithMethodSignature:self.blockSignature];
    NSInvocation *originalInvocation = info.originalInvocation;
    NSUInteger numberOfArguments = self.blockSignature.numberOfArguments;

    // Be extra paranoid. We already check that on hook registration.
    if (numberOfArguments > originalInvocation.methodSignature.numberOfArguments) {
        AspectLogError(@"Block has too many arguments. Not calling %@", info);
        return NO;
    }

    // The `self` of the block will be the AspectInfo. Optional.
    if (numberOfArguments > 1) {
        [blockInvocation setArgument:&info atIndex:1];
    }
    
	void *argBuf = NULL;
    for (NSUInteger idx = 2; idx < numberOfArguments; idx++) {
        const char *type = [originalInvocation.methodSignature getArgumentTypeAtIndex:idx];
		NSUInteger argSize;
		NSGetSizeAndAlignment(type, &argSize, NULL);
        
		if (!(argBuf = reallocf(argBuf, argSize))) {
            AspectLogError(@"Failed to allocate memory for block invocation.");
			return NO;
		}
        
		[originalInvocation getArgument:argBuf atIndex:idx];
		[blockInvocation setArgument:argBuf atIndex:idx];
    }
    
    [blockInvocation invokeWithTarget:self.block];
    
    if (argBuf != NULL) {
        free(argBuf);
    }
    return YES;
}

```

## 参考资料

[Aspects](https://github.com/steipete/Aspects)
[Runtime编程指南](https://www.jianshu.com/p/88ce7c4c2b2a)
