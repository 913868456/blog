
---
title: iOS 账号续费及证书更新流程
date:  2017-03-16 11:25
categories:
- 工具
tags:
-  证书
---

  #####  开发者账号会员快到期的时候,苹果会发送给开发者账号的注册邮箱一封邮件,提示用户账号快到期了,及时续费,一般是提前一个月提示用户续费.下面开始介绍续费流程;
1.登录开发者账号后,网页上面会有账号过期黄色提示;点击renew your membership ;会跳到支付页面;个人开发者账号是688RMB/年;
![](http://upload-images.jianshu.io/upload_images/3340896-c84af4890fcb7576.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.选择continue;
![](http://upload-images.jianshu.io/upload_images/3340896-f211f9428c8d363f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3.购买详情,选择continue,然后点击购买
 
![](http://upload-images.jianshu.io/upload_images/3340896-8cd11c2ff508d2f6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
![](http://upload-images.jianshu.io/upload_images/3340896-071e0e01c3d430cc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4.使用visa 或者 万事达信用卡支付,填写好发票信息就ok了,需要纸质发票的选择纸质发票;购买完后,苹果会发邮件告诉你续费成功.然后就是接下来的更新证书流程了.
![](http://upload-images.jianshu.io/upload_images/3340896-f65f7b4cc0565609.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 
 ###   续费后证书更新流程,首先得描述一下各个证书的定位，作用，这样在制作的时候心中有谱，对整个流程的把握也会准确一些
1、开发者证书（分为开发和发布两种，类型为[iOS](http://lib.csdn.net/base/1) Development,ios Distribution），这个是最基础的，不论是真机调试，还是上传到appstore都是需要的，是一个基证书，用来证明自己开发者身份的；
2、appID,这是每一个应用的独立标识，在设置项中可以配置该应用的权限，比如是否用到了PassBook,GameCenter,以及更常见的push服务，如果选中了push服务，那么就可以创建生成下面第3条所提到的推送证书，所以，在所有和推送相关的配置中，首先要做的就是先开通支持推送服务的appID;
3、推送证书（分为开发和发布两种，类型分别为APNs Development ios,APNs Distribution ios）,该证书在appID配置中创建生成，和开发者证书一样，安装到开发电脑上；
4、Provisioning Profiles,这个东西是很有苹果特色的一个东西，我一般称之为PP文件，该文件将appID,开发者证书，硬件Device绑定到一块儿，在开发者中心配置好后可以添加到Xcode上，也可以直接在Xcode上连接开发者中心生成，真机调试时需要在PP文件中添加真机的udid；是真机调试和必架必备之珍品；
平常我们的制作流程一般都是按以上序列进行，先利用开发者帐号登陆开发者中心，创建开发者证书，appID,在appID中开通推送服务，在开通推送服务的选项下面创建推送证书（服务器端的推送证书见下文），之后在PP文件中绑定所有的证书id,添加调试真机等；
###下面开始申请证书
 1.将原来快要过期或者已经过期的测试证书.发布证书. Provision Profile文件等跟该AppID相关的证书和PP文件revoke;
![](http://upload-images.jianshu.io/upload_images/3340896-a97820d267527d92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.创建开发者证书;
![](http://upload-images.jianshu.io/upload_images/3340896-5f43433de9e5d5bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/3340896-b2725fd7db2e92f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
选择continue,然后要求上传CSR文件,这里解释一下CSR文件,全称Certificate Signing Requst ,苹果公司要知道是谁在请求证书,需要请求者进行签名;
![](http://upload-images.jianshu.io/upload_images/3340896-f913802a5e2ca432.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
CSR文件生成方法,打开钥匙串;
![](http://upload-images.jianshu.io/upload_images/3340896-5c4407a763055aee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/3340896-177df67ecf5ed162.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
选择存储到磁盘;
 然后上传CSR文件,点击generate,就会生成开发者证书;然后下载,存到指定文件夹下,双击安装证书;
2.同理生成发布证书,同样流程创建开发推送证书和发布推送证书;选择开发推送证书类型
![](http://upload-images.jianshu.io/upload_images/3340896-1c62d3f7275ae169.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后选择appID;
![](http://upload-images.jianshu.io/upload_images/3340896-185bdbaa7f22c17a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上传CSR文件,点击generate 就生成开发环境的推送证书;
![](http://upload-images.jianshu.io/upload_images/3340896-35c8b7666789eb75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
同理生成发布环境的推送证书;
3.生成发布和推送证书后,可以去查看推送服务是否激活; 
 ![](http://upload-images.jianshu.io/upload_images/3340896-7f7a15afa3c997e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 激活后,创建Provision Profile文件,简称PP文件;
![](http://upload-images.jianshu.io/upload_images/3340896-9b55694b0e1d0467.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/3340896-fe6f70f2c6a7b15c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/3340896-ca7b60ee04269861.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/3340896-eea56e362aad9353.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/3340896-f30b174c576adeff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
填写名称,continue就ok了;同理创建发布环境PP文件;下载双击安装即可;
![](http://upload-images.jianshu.io/upload_images/3340896-f2f8121e8ae46822.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
致此,完毕.
 
