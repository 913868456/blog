
---
title: fastlane自动打包上传
date:  2019-05-08 10:56
categories:
- 工具
tags:
- fastlane
---

## Step 1
```
#安装fastlane
sudo gem install -n /usr/local/bin fastlane
```
## Step 2 
```
cd 项目目录
fastlane init
```
### Step 3
根据自己的需要配置不同的选择项

- 使用目的
 ![屏幕快照 2019-04-25 下午7.28.54.png](https://upload-images.jianshu.io/upload_images/3340896-58bf85526d9a4d31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 输入appleID 和 密码
![[图片上传中...(屏幕快照 2019-04-25 下午7.29.50.png-931c1f-1557283422808-0)]
](https://upload-images.jianshu.io/upload_images/3340896-474c8e0de8d5364c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 输入6位验证码
![屏幕快照 2019-04-25 下午7.29.50.png](https://upload-images.jianshu.io/upload_images/3340896-175c5718683cc2bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 选择y
![屏幕快照 2019-04-25 下午7.29.58.png](https://upload-images.jianshu.io/upload_images/3340896-c5f43217481f0bcf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###  Step 4
项目目录下生成fastlane文件目录
 ```
cd fastlane
vi Fastfile
```
将下面的文件内容拷贝进去,项目名称, `bundleId`, `profileName`, `workspace` ,`outputname`自己修改,
`api_key`,`user_key`自己在蒲公英内测分发 --> api 查看自己应用的 apikey和 userkey
```
default_platform(:ios)

platform :ios do
  desc "Push a new release build to the App Store"
  lane :release do
    build_app(workspace: "xxx.xcworkspace", scheme: "xxx")    # 修改为自己的项目名称
    upload_to_app_store
  end
  lane :beta do
  build_ios_app(
  workspace: "xxx.xcworkspace",  # 替换成自己的项目名
  configuration: "Release",
  scheme: "HotTravel",
  silent: true,
  clean: true,
  export_method: "ad-hoc",
  export_options: {
      provisioningProfiles: {
          "bundle ID" => "adhoc profile name"   # 修改为自己的adhoc profile文件名及buildle ID
       }
  },
  output_directory: "./build", 
  output_name: "xxx.ipa",     
  sdk: "iOS 12.2"        # use SDK as the name or path of the base SDK when building the project.
)
  pgyer(api_key: "xxxxxxxxxx", user_key: "xxxxxxxxxx")   # 自己在蒲公英内测分发 --> api 查看自己应用的 apikey和 userkey
  end
end
```

##### 打包上传
- 蒲公英 ad-hoc 
```
fastlane add_plugin pgyer  //添加蒲公英插件
fastlane init  //添加完一定要初始化,否则上传不会成功
```
⚠️ 注意
添加完蒲公英插件一定要初始化,否则上传不会成功!
添加完蒲公英插件一定要初始化,否则上传不会成功!
添加完蒲公英插件一定要初始化,否则上传不会成功!

ad-hoc包
```
fastlane beta  //ad-hoc包自动分发到蒲公英
```

- Release 
```
fastlane release

```
⚠️ 注意
1.一定要有Xcode的证书或者p12文件,否则,打包成功后上传会失败!
2.打发布包第一次的时候需要输入 apple App专用密码,登录设置 [Apple account manage]([https://appleid.apple.com/account/manage](https://appleid.apple.com/account/manage)
)
![屏幕快照 2019-08-06 下午4.08.45.png](https://upload-images.jianshu.io/upload_images/3340896-0bf9efb6bcd739a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 参考资料
[fastlane docs](https://docs.fastlane.tools/)
[蒲公英文档中心](https://www.pgyer.com/doc/view/fastlane)

