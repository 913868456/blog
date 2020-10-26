---
title:  CoreBluetooth 编程指南
date: 2018-01-03 19:02
categories:
- iOS
tags: 
- CoreBluetooth 
---
## 介绍

Core Bluetooth 框架给iOS和Mac apps 提供与外部蓝牙设备交互的能力.例如,您的app能够发现,扫描,并且跟外部设备交互.比如心率计和数码恒温计.同样 Mac 和 iOS 设备也可以作为低功耗外部设备,给其他设备提供数据.

在Core Bluetooth 中,中心设备和外部设备作为主要参与者.通常情况下我们通过一个app来实现中心设备角色,Core Bluetooth 也可以将我们的本地设备作为外部设备角色来实现.

### iOS App 状态影响蓝牙行为

当app处于后台或者挂起状态时,这种情况会影响蓝牙相关功能.默认情况下,app在后台或者挂起状态时,不能执行蓝牙任务.如果想要app在后台执行蓝牙任务,可以声明该app支持Core Bluetooth后台执行模式中的一项或者两项都支持.当app处于后台状态时,同一蓝牙任务的操作是不同的.在设计app时,要考虑这些差异.

即使app支持后台操作,当内存不足时系统会随时终止后台应用,来给前台运行的app提供内存空间.iOS 7后,Core Bluetooth支持保存中心设备和外部设备管理者对象的状态.可以使用该特性去支持长时间调用蓝牙设备的操作.

### 遵循最佳实践来提高用户体验

由于无线广播会给设备电池造成不利影响.因此,设计app的时候尽可能减少无线广播的使用.遵循最佳实践来减少该方面的不利影响同时提高用户的体验.


## Core Bluetooth 概述

本章节主要介绍开始使用Core Bluetooth 开发时需要了解的专业词汇和概念.

> 重要: 一个 iOS app 在 iOS 10.0 之后的版本,必须在info.plist 文件中描述需要访问的数据类型,否则会崩溃.访问外部设备的指定数据,在info.plist 文件中必须包含 NSBluetoothPeripheralUsageDescription.

### 中心设备和外部设备,以及他们在蓝牙通信中的角色

中心设备和外部设备是低功耗蓝牙通信中的来个主要参与者.中心设备使用来自与外部设备的信息去完成某项特定任务.下面的例子中,表示一个mac 或者 iOS app 用一种对用户来讲更友好的方式来展示来自于心率检测器的信息.

![image.png](http://upload-images.jianshu.io/upload_images/3340896-c9641636941a7f71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 中心设备发现并连接外围正在广播的设备

外部设备以 advertising packet 的格式来发送广播.广播数据中包含外部设备的名称和主要功能.一个中心设备可用浏览和听取正在广播的外部设备内容.一个中心设备可以请求连接任何发现正在广播的外部设备.

![image.png](http://upload-images.jianshu.io/upload_images/3340896-5c28cac2a725ceed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 一个外部设备的数据如何构建

外部设备包含一项或者多项服务.一个服务是一个数据集合.举例子,一个心率计的服务可以用来显示来自于心率传感器的的心率数据.

服务本身由特性或者其中包含的其他服务组成.一个特性提供外部设备服务的详细信息.举例子,心率服务包含两个特性,一个特性描述设备传感器的监听位置.另外一个特性传输测量的数据.

![image.png](http://upload-images.jianshu.io/upload_images/3340896-11772f3214f18ab9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 中心设备扫描并且跟一个外部设备数据交互

一个中心设备和外部设备成功建立连接后,可以发现外部设备提供的所有服务和特性.
中心设备可以通过读写服务特性的值来跟外部设备的服务进行交互.举例来说,app 请求当前房间的室温.或者提供给恒温计一个值来设定室温.

### 中心设备,外部设备,以及外部设备数据如何表示

本地中心设备管理者通过 CBCentralManager 对象表示.这个对象用来管理远程外部设备的遍历,发现,连接正在发送广播的外部设备.

![Core Bluetooth objects on the central side](http://upload-images.jianshu.io/upload_images/3340896-cc9e991217247c05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

远程外部设备 用CBPeripheral对象来表示, 它的数据通过 CBService 和 CBCharacteristic 对象来表示.
![image.png](http://upload-images.jianshu.io/upload_images/3340896-0fd143eeaa33aada.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

外部设备管理者 用CBPeripheralManager对象来表示. 这些对象用来管理发布的服务和相关特性.并且通过广播来给远程中心设备(CBCentral 对象)发送服务. Peripheral manager对象也用来响应远程中心设备的读写请求.
![image.png](http://upload-images.jianshu.io/upload_images/3340896-3b76d790666594fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

外部设备管理者(CBPeripheralManger)的数据通过CBMutableService 和 CBMutableCharacteristic 对象来表示.
![A local peripheral's tree of services and characteristics](http://upload-images.jianshu.io/upload_images/3340896-6bb597fc20300ec7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 执行中心角色任务

本节学习内容:

- 启动一个中心设备管理者对象
- 发现并链接正在广播的外部设备
- 扫描链接的外部设备数据
- 给一个外部设备服务特性值发送读写请求
- 订阅一个特性值的变动通知

### 启动一个中心设备管理者

创建CBCentralManager
```
myCentralManager = [[CBCentralManager alloc] initWithDelegate: self queue:nil options:nil];
```
self 必须实现 centralMangerDidUpdateState: 代理方法.更多内容请看[CBCentralManagerDelegate Protocol Reference](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate)

### 发现正在广播的外部设备

```
[myCentralManager scanForPeripheralsWithServices:nil options:nil];
```

> 注意: 如果给第一个参数指定为nil,central manager返回所有发现的外部设备.在真实情况中,一般会指定一个CBUUID对象组成的数组,每个对象表示一个服务的唯一标识.当指定好UUID对象数组后,central manager 仅返回正在广播相关服务的外部设备.浏览最感兴趣的外部设备.
UUIDs,和CBUUID对象详细内容请看[Services and Characteristics Are Identified by UUIDs](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonPeripheralRoleTasks/PerformingCommonPeripheralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH4-SW8)

central manager 每次发现外部设备,就会调用代理对象的  cetralManager:didDiscoverPeripheral:advertisementData:RSSI: 方法.新发现的外部设备以 CBPeripheral 对象返回.如果打算链接发现的外部设备,给该对象一个强引用,系统就不会释放它.

```
- (void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary *)advertisementData RSSI:(NSNUmber *)RSSI {
    NSLog(@"Discovered %@", peripheral.name);
    self.discoverdPeripheral = peripheral;
    ...
```

如果希望链接多个设备,需要用一个NSArray来存储发现的外部设备.一旦所有想连接的设备都被发现,停止浏览其他设备来节省电量:

```
[myCentralManager stopScan];
```

### 链接发现的外部设备

```
[myCentralManager connectPeripheral:peripheral options:nil];
```

如果链接请求成功,central manager 调用代理对象的centralManager:didConnectPeripheral:方法.在跟外部设备交互前,设置它的代理来确保代理接受到了合适的回调:

```
- (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral {
    
    NSLog(@"Peripheral connected");
    peripheral.delegate = self;
    ...
```

### 发现链接外部设备的服务

跟一个外部设备建立连接后,可以浏览它的数据.首先浏览外部设备提供的可用服务.由于外部设备发送广播的数据限制,会发现一个外部设备提供的服务比广播的更多.可以使用discoverServices: 方法来发现一个外部设备提供的所有服务.

```
[peripheral discoverServices:nil];
```
>注意: 在一个app中,一般不给参数传nil,因为这样会返回所有的可用服务.由于一个外部设备有许多不相关的服务,发现所有可用服务不仅浪费电量而且浪费时间.因此,指定服务的UUIDs来发现指定的服务.

当指定的服务发现后,连接的外部设备(CBPeripheral对象)调用代理对象的 peripheral:didDiscoverServices: 方法.Core Bluetooth 创建一个CBService数组---

```

- (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSerror *)error{
    
    for (CBService *service in peripheral.services) {
        
        NSLog(@"Discovered service %@", service);
        ...
    }
    ...
```

### 发现一个服务的特性

```
NSLog (@"Discovering characteristics for service %@", interestingService);
[peripheral discoverCharacteristics:nil forService:interestingService];
```

> 注意: 真实环境中,第一个参数一般不传nil,通常指定想要找的特性UUIDs

当指定服务的特性被发现后,外部设备调用代理对象的 peripheral:didDiscoverCharacteristicsForService:error: 方法.Core Bluetooth创建一个包含CBCharacteristic对象的数组来包含所有发现的特性.

```
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(NSError *)error {
    
    for (CBCharacteristic *characteristic in service.characteristics) {
        
        NSLog(@"Discovered characteristic %@", characteristic);
        ...
    }
    ...
```
### 获取一个特性的值

直接读取特性的value或者订阅它来获取特性的值.

#### 读取特性值

```
NSLog(@"Reading value for characteristic %@", interestingCharacteristic);
[peripheral readValueForCharacteristic:interestingCharacteristic];
```

当尝试读取特性值时,外部设备调用代理对象的 peripheral:didUpdateValueForCharacteristic:error:方法来获取特性值.

```
- (void)peripheral: (CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
    
    NSData *data = characteristic.value;
    //parse the data as needed 
    ...
```

> 注意:不是所有的特性都是可读的.通过CBCharacteristicPropertyRead属性常量来检测特性是否可读.如果不可读,会返回适当的错误.

#### 订阅一个特性值

```
[peripheral setNotifyValue:YES forCharacteristic: interestingCharacteristic];
```

```
当订阅失败时,可以实现下面方法来获取订阅失败原因.
- (void)peripheral:(CBPeripheral *)peripheral didUpdateNotificationStateForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
    
    if (error) {
        
       NSLog(@"Error changing notification state: %@", [error localizedDescription]) ;
    }
    ...
```

> 注意:不是所有的特性都可订阅.可以通过属性的CBCharacteristicPropertyNotify或者CBCharacteristicPropertyIndicate常量来检测属性是否可被订阅.

### 给一个特性写入值

```

NSLog(@"Wrriting value for characteristic %@", interestingCharacteristic);
[peripheral writeValue:dataToWrite forCharacteristic:interestingCharacteristic type:CBCharacteristicWriteWithResponse];

```
代理对象调用下面方法,通过CBCharacteristicWriteWithResponse来告诉app写入是否成功.
```
- (void)peripheral:(CBPeripheral *)peripheral
didWriteValueForCharacteristic:(CBCharacteristic *)characteristic
             error:(NSError *)error {
 
    if (error) {
        NSLog(@"Error writing characteristic value: %@",
            [error localizedDescription]);
    }
    ...
```

> 注意:特性仅支持特定的写入类型.可以通过一个特性的CBCharacteristicPropertyWriteWithoutResponse 或者 CBCharacteristicPropertyWrite 常量来检查支持的写入类型.

## 执行外部角色任务

本节学习内容:

- 启动一个外部设备管理对象
- 设置本地外部设备的服务和特性
- 给设备的本地数据库发布服务和特性
- 广播服务
- 响应一个连接的中心设备的读写请求
- 给订阅的中心设备发送更新后的特性值


### 启动一个外部设备管理者

```
myPeripheralManager = [[CBPeripheralManager alloc] initWithDelegate:self queue:nil options: nil];
```

创建CBPeripheralManager对象后 self 必须实现 peripheralMangagerDidUpdateState: 方法.  详细内容请看 [CBPeripheralManagerDelegate Protocol Reference]() 

### 设置外部设备管理者的服务和特性

一个本地外部设备服务和特性的数据库以类似于树状的方式组成.必须以这种方式去组织本地外部设备的服务和特性. 在执行这些任务中首先要理解如何识别这些服务和特性.

#### 服务和特性通过UUIDs来识别

iOS中通过CBUUID对象来表示外部设备指定的服务和特性.比如,128位的UUID心率服务可以使用CBUUID 对象的 UUIDWithString 方法来表示预定义的16位UUID.

```
CBUUID *heartRateServiceUUID = [CBUUID UUIDWithString: @"180D"];
```

当从一个预定义的16位UUID中创建一个CBUUID对象时,Core Bluetooth 会用Bluetooth base UUID 预填128位UUID的剩下的部分.

#### 创建自有的UUIDs来自定义服务和特性

使用命令行实用工具 uuidgen 很容易生成一个128位的UUIDs.

```
$ uuidgen 
71DA3FD1-7E10-41C1-B16F-4430B506CDE7
```
然后使用这个UUID去创建一个CBUUID对象
```
CBUID *myCustomServiceUUID = [CBUUID UUIDWithString:@"71DA3FD1-7E10-41C1-B16F-4430B506CDE7"];
```

#### 创建服务和特性树

```
myCharacteristic = [[CBMutableCharacteristic alloc] initWithType:myCharacteristicUUID properties:CBCharacteristicPropertyRead Value: myValue permissions:CBAttributePermissionsReadable];
```
> 注意:如果指定一个特性值.该值会缓存并且它的属性和许可被设置为可读.因此,如果你需要一个特性值可写,或者期望发布的服务特性能够被改变.必须指定value为nil.

现在已经创建了一个可变特性,可以创建一个可变服务跟其相关联.按照下面方法做

```
myService = [[CBMutableService alloc] initWithType:myServiceUUID primary:YES];
myService.characteristics = @[myCharacteristic];
```
### 发布服务和特性

```
[myPeripheralManager addService:myService];
```

当调用这个方法去发布服务时,外部设备管理者会通过他的代理对象调用 peripheralManager:didAddService:error: 方法.如果过发生错误或者无法发布服务,则实现该代理方法来获取错误原因.

```
- (void)peripheralManager:(CBPeripheralManager *)peripheral
            didAddService:(CBService *)service
                    error:(NSError *)error {
 
    if (error) {
        NSLog(@"Error publishing service: %@", [error localizedDescription]);
    }
    ...
```

> 当给外部设备的数据库发布一个服务和相关特性后,服务被缓存并且无法修改.


### 广播服务

```
[myPeripheralManager startAdvertising:@{ CBAdvertisementDataServiceUUIDsKey : @[myFirstService.UUID, mySecondService.UUID]}];
```

当在本地外部设备发送广播时,外部设备管者的代理对象调用 peripheralManagerDidStartAdvertising:error: 方法.如果出现错误或者服务无法广播.实现该方法来获取错误信息.

```
- (void)peripheralManagerDidStartAdvertising:(CBPeripheralManager *)peripheral
                                       error:(NSError *)error {
 
    if (error) {
        NSLog(@"Error advertising: %@", [error localizedDescription]);
    }
    ...
```

一旦广播数据开始,远程中心设备能够发现并和本地外部设备建立连接.

### 响应中心设备的读写请求

```
- (void)peripheralManager:(CBPeripheralManager *)peripheral didReceiveReadRequest:(CBATTRequest *)request {
    
    if ([request.characteristic.UUID isEqual:myCharacteristic.UUID]) {
        ...
```

如果特性的UUID匹配,下一步确定请求的索引位置是否超出特性值的边界.

```
    if (request.offset > myCharacteristic.value.length) {
        
        [myPeripheralManager respondToRequest:request withResult:CBATTErrorInvalidOffset];
        
        return;
    }
```

加入请求的offset合格.现在给请求特性属性赋值.

```
    request.value = [myCharacteristic.value subdataWithRange:NSMakeRange(request.offset, myCharacteristic.value.length - request.offset)];
```

设置好值后,响应远程中心设备请求.

```
    [myPeripheralManager respondToRequest:request withResult:CBATTErrorSuccess];
    ...
```

每调用 peripheralManager:didReceiveReadRequest: 方法一次,就会调用 respondToRequst:withResult: 一次.

> 注意: 如果特性UUID不匹配或者无法完成读请求.调用 respondToRequest:withResult: 方法,并返回合适的错误原因.(CBATTError Constants)

处理一个写入请求,直接将CBATTRequest 对象的值写入特性.

```
    myCharacteristic.value = request.value;
```

传入的写入请求数组中包含CBATTRequest 对象.每个对象写入特性都会调用  respondToRequest:withResult: 方法.

```
    [myPeripheralManager respondToRequest:[request objectAtIndeex:0] withResult:CBATTErrorSuccess];
```

### 给订阅的中心设备发送更新的特性值

中心设备订阅通知后会调用该方法

```
- (void)peripheralManager:(CBPeripheralManager *)peripheral central:(CBCentral *)central didSubscribeToCharacteristic:(CBCharacteristic *)characteristic {
    
    NSLog(@"Central subscribed to characteristic %@", characteristic);
    ...
```

```
    NSData *updatedValue = //fetch the characteristic's new value 
    
    BOOL didSendValue = [myPeripheralManager updateValue:updatedValue forCharacteristic:characteristic onSubscribedCentrals:nil];
```

## iOS Apps 中 Core Bluetooth 的后台处理

对于iOS apps,了解app在后台还是前台运行非常重要.一个app在后台的行为必须不同于前台.因为系统提供的资源是有限的.对于在iOS上所有的后台操作讨论,请看[Background Execution](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html#//apple_ref/doc/uid/TP40007072-CH4) in [App Programming Guide for iOS](https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007072)

默认情况下,许多普通的Core Bluetooth 任务---中心设备和外部设备都有--- 在后台或者挂起模式下都是禁用的.也就是说,您需要声明您的app去支持Core Bluetooth后台执行模式,从而允许您的app能够从挂起状态中唤醒去处理某些蓝牙相关的事件.即使您的app不需要所有范围内的后台处理支持.当重要事件发生后仍然可以要求系统提示.

即使您的app支持一种或者两种Core Bluetooth 后台执行模式,但是它也不能永远运行. 某些情况下,系统会中为了给前台运行的app释放内存从而终止您的app.从而使app激活的或者添加的链接丢失.

### 只在前台使用Core Bluetooth的Apps

绝大多数apps,除非您请求后台执行特定任务的许可,否则您的app短暂进入后台状态后就会转到挂起状态.在挂起状态下,您的app无法执行蓝牙相关任务.也不能感知任何蓝牙相关事件,直到它恢复到前台运行.

在中心设备这边,只在前台执行的apps---那些没有声明支持Core Bluetooth后台执行模式任何一种的app,在后台状态或者挂起状态是无法浏览或者发现正在广播的外部设备. 在外部设备这边,当一个仅支持前台模式的app处于挂起状态时，所有蓝牙相关的事件发生时将通过系统添加到队列。并且当app恢复到前台时发送给它。也就是说，Core Bluetooth 提供一种方式，当某些中心设备角色事件发生时提醒用户。用户然后能够利用这些提示来决定是否一个特殊事件许可app返回到前台。
您可以在调用CBCentralManager 类的 connectPeripheral:options: 方法使用下列的外部设备链接可选项，利用这些提示来链接一个远程外部设备。
- CBConnectPeripheralOptionNotifyOnConnectionKey
- CBConnectPeripheralOptionNotifyOnDisconnectionKey
- CBConnectPeripheralOptionNotifyOnNotificationKey

#### Core Bluetooth 后台执行模式

当需要app在后台执行一些蓝牙相关任务时，必须在info.plist 文件中声明它支持一种蓝牙后台执行模式。
info.plist 中文件 添加 key 为 UIBackgroundModes, 然后给一个数组中添加下面的字符串：
- bluetooth-central
- bluetooth-peripheral

#### 蓝牙-中心设备后台执行模式

记住在后台浏览外部设备的操作跟在前台时不同的。
- CBCentralManagerScanOptionAllowDuplicatesKey 浏览可选键 被忽略，并且多个正在广播的外部设备发现合并成一个发现事件。
- 如果所有正在扫描外部设备的app都在后台，中心设备的内部扫描广播的包也相继增长。结果，发现一个正在广播的外部设备的时间也会变长。
这些变动帮助最小化无限广播的使用,并且提高电池使用寿命。

#### 蓝牙-外部设备 后台执行模式
当app处于后台状态 时，广播跟前台是不同的：
- CBAdvertisementDataLocalNameKey 被忽略，本地外部设备的名字也不会被广播。
- 所有包含在CBAdvertisementDataServiceUUIDsKey对应值下的服务UUIDs被放置在一个特殊的“溢出”区。只有一个iOS设备明确要求扫描特它们时，才能被发现。
- 如果所有正在广播的apps在后台状态，外部设备发送广播的频率会降低。

### 更好地使用后台执行模式

由于处理蓝牙相关事件会使用无线电，无线电的使用会给电池寿命带来负面影响
最小化在后台状态下的工作量。app被唤醒的蓝牙相关事件应该尽快处理以便它能再次挂起。
任何声明支持核心蓝牙后台执行模式的app 必须遵循一些原则：
- Apps 应该基于会话类 并且提供一个接口允许用户决定何时开始和结束蓝牙相关事件的发送。
- 在被唤醒状态下，一个app有大约10秒时间去处理一项任务。 理想情况下，应该尽快处理任务并且允许再次进入后台。Apps 在后台执行过长时间会被系统限制或者杀死。
- Apps 不应该将唤醒作为一个机会去处理跟唤醒不相关的任务。

### 在后台执行长时间操作

#### 状态保存和恢复
对于一个给定的CBCentralManager对象， 系统记录下面内容：
- 中心管理者的当时正在搜索的服务（并且包含开始扫描时所有指定扫描的可选项）
- 中心管理者当时正在尝试连接或者已经连接的外部设备
- 中心管理者当时订阅的特性

对于CBPeripheralManager对象，系统记录下面内容：
- 外部设备管理者当时正在广播的数据
- 外部设备管理者发布给设备数据库的服务
- 订阅外部设备服务特性值的中心设备。

在创建中心管理者对象的时候就需要添加状态保存和恢复策略
```
   myCentralManager = [[CBCentralManager alloc] initWithDelegate:self queue:nil options:@{ CBCentralManagerOptionRestoreIdentifierKey : @“myCentralManagerIdentifer”}];
```
外部设备管理者使用CBPeripheralManagerOptionRestoreIdentiferKey 来初始化。

app重新启动时，可以获取所有系统保存的唯一标识

```
- (BOOL)application:(UIApplication *)application
didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
 
    NSArray *centralManagerIdentifiers =
        launchOptions[UIApplicationLaunchOptionsBluetoothCentralsKey];
    ...
```

#### 实现合适的恢复代理方法

> 重要:对于存储状态并需要恢复状态的app,当app重启进入后台处理蓝牙相关事件时,先调用 centralManager:willRestoreState: 和 peripheralManager:willRestoreState: 方法.如果没有保存状态,则调用 centralManagerDidUpdateState: 和 peripheralManagerDidUpdateState: 代理方法.

```
- (void)centralManager:(CBCentralManager *)central willRestoreState:(NSDictionary *)state{
    
    NSArray *peripherals = state[CBCentralManagerRestoredStatePeripheralKey];
    ...
```

#### 更新初始化进程

```
    NSUInteger serviceUUIDIndex =
        [peripheral.services indexOfObjectPassingTest:^BOOL(CBService *obj,
        NSUInteger index, BOOL *stop) {
            return [obj.UUID isEqual:myServiceUUIDString];
        }];
 
    if (serviceUUIDIndex == NSNotFound) {
        [peripheral discoverServices:@[myServiceUUIDString]];
        ...
```

## 中心设备和远程外部设备交互最佳实践

### 注意广播的使用和电量消耗

尽可能最小化广播的使用.因为无线广播会给iOS设备硬件的电池寿命造成负面影响.

- 只有需要的时候才扫描设备
- 只有必要时才去指明CBCentralManagerScanOptionAllowDuplicatesKey Option
- 正确地浏览外部设备数据
```
    [peripheral discoverServices:@[firstServiceUUID, secondServiceUUID]];
```
查找服务特性也同样适用这种方式.
- 订阅经常变动的特性值
- 所有需求数据满足时取消设备连接
```
    [myCentralManager cancelPeripheralConnection:peripheral];
```

### 外部设备重连

使用下面三种方式去重连外部设备:

- 取回已知外部设备列表 
使用 retrievePeripheralsWithIdentifiers: 方法.
- 取回当前系统连接的外部设备列表 
使用 retriveConnectedPeripheralsWithServices: 方法.如果要查找的外部设备在列表中,连接它.
- 扫描并查找外部设备. 
使用 scanForPeripheralsWithServices:options 方法.如果找到.连接它.

下面是一个重连的流程示例:
![重连流程](http://upload-images.jianshu.io/upload_images/3340896-6d4adcc223f2f43a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 设置本地设备作为外部设备最佳实践

### 广播时要考虑的内容 

遵循广播数据限制.
当创建一个广播字典时,只能指定 CBAdvertisementDataLocalNameKey, CBAdvertisementDataServiceUUIDsKey.如果指定其他key会报错.
同样广播数据占用的空间也要限制.至多使用28字节的空间来初始化广播数据.

仅在需要的时候去广播数据,使用下面方法来停止广播
```
    [myPeripheralManager stopAdvetising];
```
让用户来决定何时广播
### 配置外部设备特性

下面两个分组帮助我们在需要执行以下任务是提供一些指导:
- 允许链接的中心设备订阅您的特性
- 保护敏特性值,防止未配对中心设备的访问

配置特性来支持通知
```
myCharacteristic = [[CBMutableCharacteristic alloc] initWithType:myCharacteristicUUID properties:CBCharacteristicPropertyRead | CBCharacteristicPropertyNotify value:nil permissions:CBAttributePermissionsReadable];

```

要求一个配对设备来访问敏感数据
```
    emailCharacterristic = [[CBMutableCharacteristic alloc] initWithType:emailCharacteristicUUID properties:CBCharacteristicPropertyRead | CBCharacteristicPropertyNotifyEncryptionRequired value:nil permission:CBAttributePermissionsReadEncryptionRequired];

```

# 项目Demo
[蓝牙4.0 Core Bluetooth Demo](https://github.com/913868456/OCDemo)
上面链接是 **Core Bluetooth** 编程的Demo，能够对**Core Bluetooth** 有一个基本的了解。本项目使用一个蓝牙手环作为测试设备，读取蓝牙手环内的一些服务。如果想要使用其他蓝牙设备运行项目，把 **"MH08"** 替换为自己的设备名称前缀即可。
```
 if ([peripheral.name hasPrefix:@"MH08"] ) {
        self.bandPeripheral = peripheral;        //强引用外部设备对象,否则会释放
        [self.centralManager stopScan];          //发现指定外设后,为了保护电池寿命和节约电量,中心管理者停止扫描
        NSLog(@"链接外部设备: %@", peripheral.name);
        [self.centralManager connectPeripheral:peripheral options:nil];
    }
```
# GitHub 优秀开源
由于使用官方提供的API编程特别凌乱，所以Github上有优秀开发者对其进行了封装。
详情请看[BabyBluetooth ](https://github.com/coolnameismy/BabyBluetooth)

# 参考资料
[Core Bluetooth Programming Guide](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/AboutCoreBluetooth/Introduction.html#//apple_ref/doc/uid/TP40013257-CH1-SW1)
