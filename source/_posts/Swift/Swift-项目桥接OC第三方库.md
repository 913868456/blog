---
title:  Swift Briging Objective-C
categories:
- Swift
tags: 
- Swift
hide: true

---

1. 创建Header文件
 ![创建桥接文件](https://upload-images.jianshu.io/upload_images/3340896-4d064cca705440a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. ```Targets->Build Setting```选中搜索栏,搜索```Swift compiler```
![配置桥接文件路径](https://upload-images.jianshu.io/upload_images/3340896-f4e4a89033693caf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3. ```Objective-C Bridging Header```选项后输入桥接文件路径,如果在根目录,直接输入桥接文件名

4. 添加需要的```Objective-C```框架
![添加框架](https://upload-images.jianshu.io/upload_images/3340896-5d6adf47bd8106ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




