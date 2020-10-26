
---
title:  Apple登录集成
date:  2020-03-30 14:00
categories:
- iOS
tags: 
- iOS 
---
## 配置

1.target -> Signing & Capabilities, 点击加号, 搜索 Sign In with Apple

2.General -> Frameworks, Libraries, and Embedded Content,点击加号， 搜索 AuthenticationServices 并导入进去， Embed 选择  Embed & Sign


## 登录

- 应用启动登录校验
```
 func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        let appleIDProvider = ASAuthorizationAppleIDProvider()
        //此处Userid可以为本地存储的token或其他登陆确认身份内容
        appleIDProvider.getCredentialState(forUserID: KeychainItem.currentUserIdentifier) { (credentialState, error) in
            switch credentialState {
            case .authorized:
                //授权有效，走登陆成功页面
                break
            case .revoked, .notFound:
                // 走登陆页面
                DispatchQueue.main.async {
                    
                }
            default:
                break
            }
        }
        return true
    }
```

- 创建appid登录按钮并添加到登陆页面指定位置

```
    let authorizationButton = ASAuthorizationAppleIDButton()
 
    @objc
    func handleAuthorizationAppleIDButtonPress() {
        let appleIDProvider = ASAuthorizationAppleIDProvider()
        let request = appleIDProvider.createRequest()
        request.requestedScopes = [.fullName, .email]
        
        let authorizationController = ASAuthorizationController(authorizationRequests: [request])
        authorizationController.delegate = self
        authorizationController.presentationContextProvider = self
        authorizationController.performRequests()
    }

```

- 处理登陆事件代理
```
extension LoginViewController: ASAuthorizationControllerDelegate {
    /// - Tag: 认证状态回调信息
    func authorizationController(controller: ASAuthorizationController, didCompleteWithAuthorization authorization: ASAuthorization) {
        switch authorization.credential {
        case let appleIDCredential as ASAuthorizationAppleIDCredential: //此处为appleID 认证
            
            // 根据以下信息在后端创建账户
            let userIdentifier = appleIDCredential.user  //用户唯一标志符
            let fullName = appleIDCredential.fullName
            let email = appleIDCredential.email
            
            // 后端请求注册账户及密码及登录成功后存储登录信息处理（这里根据业务情况来决定是否重新注册用户信息）
            
            // 成功后进入登陆后页面
    
        case let passwordCredential as ASPasswordCredential: //此处为账号密码认证
        
            // Sign in using an existing iCloud Keychain credential.
            let username = passwordCredential.user
            let password = passwordCredential.password
            
            // 做一些账号密码的登录处理
            
            // 成功后进入登陆后页面
        default:
            break
        }
    }
    
    /// - Tag: 获取认证状态失败后的错误信息
    func authorizationController(controller: ASAuthorizationController, didCompleteWithError error: Error) {
        // Handle error.
    }
}
```

- 处理视图代理
```
extension LoginViewController: ASAuthorizationControllerPresentationContextProviding {
    /// - Tag: 提供presention视图
    func presentationAnchor(for controller: ASAuthorizationController) -> ASPresentationAnchor {
        return self.view.window!
    }
}

```

- 进入登录视图进行登陆验证 （此处可根据业务情况来决定是否添加）
```
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        performExistingAccountSetupFlows()
    }
    
    func performExistingAccountSetupFlows() {
        // Prepare requests for both Apple ID and password providers.
        let requests = [ASAuthorizationAppleIDProvider().createRequest(),
                        ASAuthorizationPasswordProvider().createRequest()]
        
        // Create an authorization controller with the given requests.
        let authorizationController = ASAuthorizationController(authorizationRequests: requests)
        authorizationController.delegate = self
        authorizationController.presentationContextProvider = self
        authorizationController.performRequests()
    }
```

## 退出登录

- 根据`userIdentifier`退出登陆流程

##参考资料
[Implementing User Authentication with Sign in with Apple](https://developer.apple.com/documentation/authenticationservices/implementing_user_authentication_with_sign_in_with_apple)
