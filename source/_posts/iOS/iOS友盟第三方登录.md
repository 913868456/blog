
---
title:  友盟三方登录 
date:  2017-05-16 11:22
categories:
- iOS
tags: 
- 友盟 
---
# API
   - 绑定账号
   -  解绑账号
   - 校验UID
   - 校验手机号

---
# 登录流程

![登录流程.png](http://upload-images.jianshu.io/upload_images/3340896-3c9abdd048dbc676.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
### 注意:
- 友盟SDK需要及时更新,4.x,5.x版本可能无法获取到uid
- QQ第三方登录需要向腾讯开发者平台申请QQ统一UID,然后通过腾讯给的接口请求UID;否则,iOS和Android获取的uid是不一样的
- 微信第三方登录功能需要登录微信开发者中心,开通第三方登录授权 (付费功能)
- 如果集成了友盟分享功能,集成第三方登录只需获取用户信息即可

// 在需要进行获取登录信息的UIViewController中加入如下代码
```
#import <UMSocialCore/UMSocialCore.h>

- (void)getUserInfoForPlatform:(UMSocialPlatformType)platformType
{
    [[UMSocialManager defaultManager] getUserInfoWithPlatform:platformType currentViewController:self completion:^(id result, NSError *error) {

        UMSocialUserInfoResponse *resp = result;

        // 第三方登录数据(为空表示平台未提供)
        // 授权数据
        NSLog(@" uid: %@", resp.uid);
        NSLog(@" openid: %@", resp.openid);
        NSLog(@" accessToken: %@", resp.accessToken);
        NSLog(@" refreshToken: %@", resp.refreshToken);
        NSLog(@" expiration: %@", resp.expiration);

        // 用户数据
        NSLog(@" name: %@", resp.name);
        NSLog(@" iconurl: %@", resp.iconurl);
        NSLog(@" gender: %@", resp.gender);

        // 第三方平台SDK原始数据
        NSLog(@" originalResponse: %@", resp.originalResponse);
    }];
}
```
