
---
title:  JavaScriptCore与WebView 
date: 2017-11-07 21:57
categories:
- iOS
tags: 
- JavaScriptCore
- WebView
---
### Html中JS 与 Native交互情况

- Native调用JS
- JS调用Native

### 一.  获取WebView的JSContext

```
    func webViewDidFinishLoad(_ webView: UIWebView) {
        
        self.context = webView.value(forKeyPath: "documentView.webView.mainFrame.javaScriptContext") as? JSContext
    }
```

> 不同网页JSContext不同,只有在 webViewDidFinishLoad() 方法中获取,才能获取当前网页的JSContext


### 二.   Native调用JS

Html中JS提供的接口

```
<script>
	
	  function handler(argument) {

        document.getElementById("jsContent").innerHTML = argument["name"] + " " + argument["age"] + "岁 " +  argument["job"]

      }
</script>

```
Native调用方法

```
     //本地调用JS方法
    @objc func nativeCallJS() {

        //方法一 (直接执行JS脚本)
        let scriptStr = "handler({'name':'小明', 'age':'18','job':'学生'})"
        let _ = self.context?.evaluateScript(scriptStr)
        
        //方法二 (获取函数,然后调用函数传参)
        let handleFunc = self.context?.objectForKeyedSubscript("handler")
        let paraDic = ["name":"小明", "age": 18, "job":"学生"] as [String : Any]
        let _ = handleFunc?.call(withArguments: [paraDic])

    }
```
### 三.    JS调用Native 

#### 方式一: 使用Block(Swift 用闭包)

```
- (void)webViewDidFinishLoad:(UIWebView *)webView{

    //获取JS运行环境
    _context = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
    //html调用无参数OC
    _context[@"test1"] = ^(){
        [self method1];
    };
    
    _context[@"test2"] = ^(){
        
        NSArray *args = [JSContext currentArguments];//传过来的参数
        NSString *name = args[0];
        NSString *str = args[1];
        [self method2:name and:str];
    };

}

```

#### 方式二: 使用JSExport 

```
@objc protocol SwiftBridgeProtocol: JSExport{
   
    //JS调用本地方法(无参数)
    func method1()
    func method2(_ title: String, _ message: String)
    func method3(_ handlerName: String)
}

@objc class SwiftBrige: NSObject, SwiftBridgeProtocol{
    
    weak var controller: UIViewController?
    weak var context   : JSContext?
    
    //JS调用本地方法(无参数)
    func method1()  {
        
        let controller = UIAlertController.init(title: "测试", message: "JS调用本地方法", preferredStyle: .alert)
        controller.addAction(UIAlertAction(title: "取消", style: .cancel, handler: nil))
        self.controller?.present(controller, animated: true, completion: nil)
    }
    
    //JS调用本地方法(传参数)
    func method2(_ title: String, _ message: String) {
        
        let controller = UIAlertController.init(title: title, message: message, preferredStyle: .alert)
        controller.addAction(UIAlertAction(title: "取消", style: .cancel, handler: nil))
        self.controller?.present(controller, animated: true, completion: nil)
    }
    
    //JS调用本地方法  本地方法回调JS方法
    func method3(_ handlerName: String){
        
        //回调参数
        let paraDic = ["name":"小明", "age": 18, "job":"学生"] as [String : Any]
        
        //获取JS回调函数
        let handleFunc = self.context?.objectForKeyedSubscript("\(handlerName)")
        
        //调用该函数
        let _ = handleFunc?.call(withArguments: [paraDic])
    }
}
```
[项目代码Swift](https://github.com/913868456/SwiftDemo)
[项目代码OC](https://github.com/913868456/OCDemo)

