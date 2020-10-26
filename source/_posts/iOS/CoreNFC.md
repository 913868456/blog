
---
title:  CoreNFC 
date: 2017-10-30 17:23
categories:
- iOS
tags: 
- CoreNFC 
---

检测NFC 设备并读取里面包含的NDEF格式数据.

## 简述

使用CoreNFC,你能够读取(目前只能读取) 1-5种NDEF格式的Near Field Communication(NFC)设备信息.为了读取NFC信息,你的app需要创建一个NDEF reader session 并且实现相关协议.一个运行的reader session 会查询NFC设备并且返回它包含的NDEF信息时,会调用相关协议方法.代理对象能够读取相关信息并决定reader session 是否失效.

>  #### 注意:
> 读取NFC数据的设备仅支持iPhone 7 以上机型
> 当前的NFC芯片,大部分读取不到信息,必须是标准的NFCNDEF格式写入的芯片才能读取.(北京地铁卡,门禁卡,身份证等卡片测试的时候无法识别,反正我是什么都没有读到,就出现一个调取NFC的界面,然后就没有然后了...)

#### 项目配置
-  开启NFC功能(如果你的证书勾选自动管理的话, TARGETS -> Capabilities -> Near Field Communication Tag Reading 勾选ON),然后会自动创建一个 "项目名.entitlements" 文件.

![开启NFC.jpeg](http://upload-images.jianshu.io/upload_images/3340896-88ca287b76f4f852.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 证书对工程进行授权
![TestNFC.entitlements](http://upload-images.jianshu.io/upload_images/3340896-ab200d5472bdfcf5.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
- info.plist 文件中添加隐私访问权限申请.
![info.plist](http://upload-images.jianshu.io/upload_images/3340896-e6375dba9089d513.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 核心代码

```
class ViewController: UIViewController, NFCNDEFReaderSessionDelegate {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //判断设备是否支持NFC,支持创建NFC对话对象
        if NFCNDEFReaderSession.readingAvailable == true {
            
            let session = NFCNDEFReaderSession.init(delegate: self, queue: nil, invalidateAfterFirstRead: false)
            session.alertMessage = "请将卡片靠近手机"
            session.begin()
        }
    }
    
    // MARK: - NFCNDEFReaderSessionDelegate
    func readerSession(_ session: NFCNDEFReaderSession, didInvalidateWithError error: Error) {
        debugPrint("\(session)\n\(error)")
    }
    
    func readerSession(_ session: NFCNDEFReaderSession, didDetectNDEFs messages: [NFCNDEFMessage]) {
        debugPrint("\(session)\n\(messages)")
    }
}

```

## Topics

#### Reader Session  

```
class [NFCNDEFReaderSession ]()
//reader session 类  用来探测NFC设备的NDEF数据.

//实例创建

init(delegate: NFCNDEFReaderSessionDelegate, queue: DispatchQueue?, invalidateAfterFirstRead: Bool)
//参数介绍:
//delegate: 代理对象
//queue    : 代理对象回调被分派的队列.  可选值, 置为nil 后,会在内部为该会话创建一个串行队列
//invalidateAfterFirRead: 设备第一次成功读取到信息后,reader session是否自动失效. 一般填false

//设备可用性检查
//类属性,返回设备是否支持NFC读取功能
class var readingAvailable: Bool

 protocol [NFCReaderSessionProtocol]()
//通用交互接口协议  用来跟reader session进行交互

// 会话检测到NDEF信息
func readerSession(NFCNDEFReaderSession, didDetectNDEFs: [NFCNDEFMessage])

// 会话失效,返回错误信息
func readerSession(NFCNDEFReaderSession, didInvalidateWithError: Error)

class [NFCReaderSession]()
// NFCNDEFReaderSession 的基类.
```
***
#### NDEF Messages
```
class   [NFCNDEFMessage]()
//  records 属性是[NFCNDEFPayload]数组

class  [NFCNDEFPayload]()
//  NFC NDEF信息的载体

//载体的id
var identifier : Data

// 数据内容,这是我们需要读取的二进制信息
var payload: Data

// 载体类型
var type      : Data 

// 类型格式名  枚举值
var typeNameFormat: NFCTypeNameFormat

enum [NFCTypeNameFormat]()
//枚举值 标明NFC NDEF信息的格式
```
#### Errors 
```
struct [NFCReaderError]()
//结构体 reader session的错误类型
```
