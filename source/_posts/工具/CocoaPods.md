---
title: Cocoapods
date:  2017-5-18 22:39
categories:
- 工具
tags:
- 依赖管理
---

### 使用CocoaPods

#### 安装CocoaPods 
打开命令行
```
// OSX 10.11后安装CocoaPods命令
sudo gem install -n /usr/local/bin cocoapods -v 1.9.0
```
#### 项目使用CocoaPods
- 生成Podfile，进入.xcodeproj文件所在目录
```
pod init
```

- Podfile

```
platform :ios, '8.0'

target 'AppName' do

end 

```

- 引入不同的库，并指定版本

```
pod 'X', '~> 1.1'

```

- 引入非CocoaPods公共Git仓库中的库，并可以指定具体的commit, branch 或者 tag, 编译版本

```
pod 'Y', :git => 'https://github.com/NSHipster/Y.git', :commit => 'b4dc0ffee'
pod 'LookinServer', :configurations => ['Debug']

```
- 引入私有库   

```
//添加私有库地址到本地的cocopods中
$ pod repo add REPO_NAME SOURCE_URL

//Podfile文件中添加私有库源
source 'URL_TO_REPOSITORY'

```

- 安装所需要的库

```
pod install 
```
Cocoapods 会你用递归分析所有需求，最后序列化为Podfile.lock（建议将 podfile.lock和.xcworkspace 纳入Git版本管理中，该文件记录这Pod里面相关库的历史安装版本）

比如，如果两个库都需要使用AFNetworking，CocoaPods 会确定一个同时能被这两库使用的版本，然后将同一个安装版本链接到两个不同的库中。

CocoaPods 会创建一个新的包含之前安装好的静态库 Xcode 项目，然后将它们链接成一个新的 libPods.a target。你原有的项目将会依赖这个新的静态库。一个 xcworkspace 文件会被创建，从此之后，你应该只打开这个 xcworkspace 文件来进行开发。

- 升级指定库

```
pod update 'C'
```
- 升级私有库

```
pod repo update REPO_NAME
```

- 尝试使用Cocoapod

```
pod try 'D'
```

### 建立自己的CocoaPods 

#### 规范
.podspec文件作为 CocoaPods 的一个独立单元，包含了名称，版本，许可证，和源码文件等所有信息。

以下是Alamofire的podspec文件内容

```
Pod::Spec.new do |s|
  s.name = 'Alamofire'
  s.version = '5.3.0'
  s.license = 'MIT'
  s.summary = 'Elegant HTTP Networking in Swift'
  s.homepage = 'https://github.com/Alamofire/Alamofire'
  s.authors = { 'Alamofire Software Foundation' => 'info@alamofire.org' }
  s.source = { :git => 'https://github.com/Alamofire/Alamofire.git', :tag => s.version }
  s.documentation_url = 'https://alamofire.github.io/Alamofire/'

  s.ios.deployment_target = '10.0'
  s.osx.deployment_target = '10.12'
  s.tvos.deployment_target = '10.0'
  s.watchos.deployment_target = '3.0'

  s.swift_versions = ['5.1', '5.2', '5.3']

  s.source_files = 'Source/*.swift'

  s.frameworks = 'CFNetwork'
end


```

一旦把这个.podspec发布到公共数据库中，任何想使用它的开发者，只需要在 Podfile 中加入如下声明即可：

```
pod 'Alamofire', '~> 5.0'

```
.podspec文件也可以作为管理内部代码的利器：
```
pod 'Z', :path => 'path/to/directory/with/podspec'
```

#### 发布CocoaPods

CocoaPods 0.33 中加入了Trunk服务。

要想使用 Trunk 服务，首先你需要注册自己的电脑。这很简单，只要你指明你的邮箱地址（spec 文件中的）和名称即可。

```
$ pod trunk register mattt@nshipster.com "Mattt Thompson"
```
至此，你就可以通过以下命令来方便地发布和升级你的 Pod！

```
$ pod trunk push NAME.podspec

```

### 参考资料
[CocoaPods](https://guides.cocoapods.org/)

[NSHipster](https://nshipster.cn/cocoapods/)