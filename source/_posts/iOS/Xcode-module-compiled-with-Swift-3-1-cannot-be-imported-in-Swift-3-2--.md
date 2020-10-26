---
title:  Xcode问题
date: 2017-10-10 10:24
categories:
- iOS
tags: 
- Xcode 
---

##  问题描述


首先我们的项目是用Xcode 8.3版本写的,第三方库使用Carthage管理,从同事那里拷贝了一个Xcode9的包,然后安装,编译项目就出现了module compiled with Swift 3.1 cannot be imported in Swift 3.2: /User/......Alamofire.framework 这样的报错信息
 ![66C43EBF3CF24F18CCFFCABFC94025ED.png](http://upload-images.jianshu.io/upload_images/3340896-c90dc3fcea0add44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 解决方法
把旧的Xcode版本删掉,将Xcode 9拷贝到应用程序中, 然后打开终端 cd到项目目录, Carthage update --platform ios,等更新完毕后,重新打开项目, Commend + shift + k, 然后重新编译一下就不报错了.
