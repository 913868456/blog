
---
title: Carthage安装
date:  2019-05-22 22:39
categories:
- 工具
tags:
- 依赖管理
---

###  Homebrew 更新
```
brew update

# brew 有时候更新会没有反应,以下是解决方案
# 进入 brew 的仓库根目录
cd "$(brew --repo)"
# 修改为中科大的源
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-cask"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
brew update -v
```
### Carthage 更新
```
brew upgrade carthage
```
### 创建Cartfile文件
```
cd 项目根目录
touch Cartfile
```
### cartfile添加指定库
```
github "SVProgressHUD/SVProgressHUD" ~> 1.0
```
### 更新
```
carthage update --platform iOS 
```
### 添加framwork
点击"项目名称"-> "target" -> "Gerneral"，在最底部找到"Linked Frameworks and Libraries"。
### 添加run script
点击"项目名称"-> "target" -> "Build Phases', 点击 "+"-> "New Run Script Phase".
脚本内容
```
/usr/local/bin/carthage copy-frameworks
```
编辑 "Input Files"路径
```
$(SRCROOT)/Carthage/Build/iOS/Kingfisher.framework
```
编辑"Output Files"路径
```
$(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/Kingfisher.framework
```

Done!


## 参考资料
[HomeBrew 修改镜像源解决慢的问题](https://crowall.com/topic/412)
