---
title:  Router 
date: 2017-11-26 23:12
categories:
- iOS
tags: 
- Router 
---

##  需求描述
通过scheme跳转到应用指定页面

## 实现逻辑
-  定义URL，通过其获取控制器名和控制器属性参数
- 通过控制器名创建对应控制器
- 给控制器属性赋值
- 跳转到相应控制器

### [项目Demo](https://github.com/913868456/GFRouter) 

## 具体实现

1. 定义URL规则

 scheme :// host /控制器名?属性名=属性值&属性名=属性值

该方式符合标准的URL定义规范，同时也方便以后扩展

通过控制器名称获取控制器，需要用到runtime，所以需要先导入<objc/runtime.h> 头文件

```
//通过URL获取控制器
+ (UIViewController *)getControllerFromURL:(NSURL *)URL{
    
    if (URL.path.length > 1) {
        
        NSString *subPath = [URL.path substringFromIndex:1];
        UIViewController *vc = [GFRouter getControllerFromClassName:subPath];
        return vc;
    }else{
        
        return nil;
    }
}

//通过类名获取控制器
+ (UIViewController *)getControllerFromClassName:(NSString *)controllerName{
    
    const char * name = [controllerName cStringUsingEncoding:NSASCIIStringEncoding];
    
    // 从一个类名返回一个类
    Class newClass = objc_getClass(name);
    // 创建对象
    if (newClass == nil) return nil;
    return [[newClass alloc] init];
}

```

获取控制器属性参数 paraDic

```

+ (NSMutableDictionary *)getParaWith:(NSURL *)URL{
    
    NSMutableDictionary *properties = [NSMutableDictionary dictionary];
    // Extract Params From Query.
    NSArray<NSURLQueryItem *> *queryItems = [[NSURLComponents alloc] initWithURL:URL resolvingAgainstBaseURL:false].queryItems;
    
    for (NSURLQueryItem *item in queryItems) {
       properties[item.name] = item.value;
    }
    
    return properties;
}

```
2. 控制器属性赋值，使用 KVC 
>  使用 KVC 赋值的时候，需要判断将要赋值的属性是否在控制器中存在

```

//属性赋值
+ (UIViewController *)setPropertyWith:(NSMutableDictionary *)paraDic and:(UIViewController *)controller{
    
    [paraDic enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
        // 检测这个对象是否存在该属性
        if ([GFRouter checkIsExistPropertyWithInstance:controller verifyPropertyName:key]) {
            // 利用kvc赋值
            [controller setValue:obj forKey:key];
        }
    }];
    return controller;
}

+ (BOOL)checkIsExistPropertyWithInstance:(id)instance verifyPropertyName:(NSString *)verifyPropertyName
{
    unsigned int outCount, i;
    
    // 获取对象里的属性列表
    objc_property_t * properties = class_copyPropertyList([instance class], &outCount);
    
    for (i = 0; i < outCount; i++) {
        objc_property_t property =properties[i];
        //  属性名转成字符串
        NSString *propertyName = [[NSString alloc] initWithCString:property_getName(property) encoding:NSUTF8StringEncoding];
        // 判断该属性是否存在
        if ([propertyName isEqualToString:verifyPropertyName]) {
            free(properties);
            return YES;
        }
    }
    free(properties);
    
    return NO;
}

```

3. 获取当前控制器，推出目标控制器

```
//获取当前viewController
+ (UIViewController *)getCurrentVC
{
    UIViewController *result = nil;
    UIWindow * window = [[UIApplication sharedApplication] keyWindow];
    //app默认windowLevel是UIWindowLevelNormal，如果不是，找到UIWindowLevelNormal的
    if (window.windowLevel != UIWindowLevelNormal)
    {
        NSArray *windows = [[UIApplication sharedApplication] windows];
        for(UIWindow * tmpWin in windows)
        {
            if (tmpWin.windowLevel == UIWindowLevelNormal)
            {
                window = tmpWin;
                break;
            }
        }
    }
    id  nextResponder = nil;
    UIViewController *appRootVC=window.rootViewController;
    //    如果是present上来的appRootVC.presentedViewController 不为nil
    if (appRootVC.presentedViewController) {
        nextResponder = appRootVC.presentedViewController;
    }else{
        
        //        NSLog(@"===%@",[window subviews]);
        UIView *frontView = [[window subviews] objectAtIndex:0];
        nextResponder = [frontView nextResponder];
    }
    
    if ([nextResponder isKindOfClass:[UITabBarController class]]){
        UITabBarController * tabbar = (UITabBarController *)nextResponder;
        UINavigationController * nav = (UINavigationController *)tabbar.viewControllers[tabbar.selectedIndex];
        //UINavigationController * nav = tabbar.selectedViewController ; 上下两种写法都行
        result=nav.childViewControllers.lastObject;
        
    }else if ([nextResponder isKindOfClass:[UINavigationController class]]){
        UIViewController * nav = (UIViewController *)nextResponder;
        result = nav.childViewControllers.lastObject;
    }else{
        result = nextResponder;
    }
    return result;
}

//导航推出控制器
+ (void)pushWith:(UIViewController *)controller{
    
    UINavigationController *nc = [GFRouter getCurrentVC].navigationController;
    controller.hidesBottomBarWhenPushed = YES;
    
    if (nc) {
        [nc pushViewController:controller animated:YES];
    }
}

```

## 拓展： 

通过上述方式，我们还可以往URL中拼上一些参数，来操作控制器的推出方式。

比如 xxx://host:8080/控制器名?displayStyle=present

```

  //默认push推出控制器
    if ([paraDic[@"displayStyle"] isEqualToString:@"present"]) {
        [router presentWith:controller];
    }else{
        [router pushWith:controller];
    }

//模态推出控制器
- (void)presentWith:(UIViewController *)controller{
    
    UIViewController *vc = [GFRouter getCurrentVC];
    
    if (vc.class != controller.class) {
        [vc presentViewController:controller animated:YES completion:nil];
    }
```

也可在控制器内部使用router,传入控制器名称和属性键值字典，然后像上述方式一样推出控制器。具体实现就不多说了，自己可以动手操作一下。


