
---
title:  NSLayoutConstraint 
date: 2017-10-19 16:02
categories:
- iOS
tags: 
- Layout 
---
#### 创建约束的两个方法

```

方法一:
/* Create an array of constraints using an ASCII art-like visual format string.
*/
open class func constraints(withVisualFormat format: String, options opts: NSLayoutFormatOptions = [], metrics: [String : Any]?, views: [String : Any]) -> [NSLayoutConstraint]

方法二:
/* Create constraints explicitly. Constraints are of the form "view1.attr1 = view2.attr2 * multiplier + constant"
If your equation does not have a second view and attribute, use nil and NSLayoutAttributeNotAnAttribute.
*/
public convenience init(item view1: Any, attribute attr1: NSLayoutAttribute, relatedBy relation: NSLayoutRelation, toItem view2: Any?, attribute attr2: NSLayoutAttribute, multiplier: CGFloat, constant c: CGFloat)

```

#### 参数介绍
方法一:

format: VFL字符串 
options: NSLayoutformatOptins (如果有一个则只需填写一个,若有多个则需要数组包含所有选项)
metrics 以字典的形式设置距离变量
比如 "H:|-[dis1]-[view1]-[dis2]-[view2(==view1)]-20-|"这句中的[dis1] [dis2]为视图变量,将字典的view1 view2即为key 对应相应的视图
views 以字典的形式设置视图变量
比如 "H:|-20-[view1]-20-[view2(==view1)]-20-|"这句中的[view1] [view2]为视图变量,将字典的view1 view2即为key 对应相应的视图

方法二:

view1:将要设置约束的控件
view2:参考控件
attr1:  将要设置控件的属性
attr2:  参考控件的属性
multiplier: 相乘系数
constant： 约束常量

view1.attr1 = view2.attr2 * multiplier + constant

#### Visual Format Laguage(VFL)介绍

H:表示水平约束
V:表示垂直方向约束
| 表示父视图
-30- 表示间距30Point
[View] 表示UI控件View

#### Example
"H:|-20-[aView]-15-[bView(==aView)]-20-|"
翻译:  水平方向约束:距离父视图左右间距为20,aView 和 bView水平间距为15,aView和bView的宽度相等.

#### 布局视图

![1508392677456.jpg](http://upload-images.jianshu.io/upload_images/3340896-d4ba05585a04da9f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 布局代码
```
/**
创建两个视图,两个视图距离顶部50,距离左右边界20,两个视图间距15,等宽,等高,视图高度为150
*/
let aView = UIView()
aView.backgroundColor = .red
view.addSubview(aView)

let bView = UIView()
bView.backgroundColor = .brown
view.addSubview(bView)

//关闭Autoresize,防止与AutoLayout冲突,参与布局的视图都需要关闭
view.translatesAutoresizingMaskIntoConstraints = false
aView.translatesAutoresizingMaskIntoConstraints = false
bView.translatesAutoresizingMaskIntoConstraints = false

//添加两个视图的水平约束
let contraintsH = NSLayoutConstraint.constraints(withVisualFormat: "H:|-20-[aView]-15-[bView(==aView)]-20-|", options: [.alignAllTop, .alignAllBottom], metrics: nil, views: ["aView": aView, "bView": bView])
view.addConstraints(contraintsH)

//添加两个视图的垂直约束
let contraintsV = NSLayoutConstraint.constraints(withVisualFormat: "V:|-50-[aView(150)]", options: .init(rawValue: 0), metrics: nil, views: ["aView": aView])
view.addConstraints(contraintsV)

/**
aView内添加一个子视图cView,cView与aView的中心对齐,宽高是aView的一半
*/
let cView = UIView()
cView.backgroundColor = .yellow
aView.addSubview(cView)

//关闭autoresize
cView.translatesAutoresizingMaskIntoConstraints = false

//cView添加约束
let constraintX = NSLayoutConstraint.init(item: cView, attribute: .centerX, relatedBy: .equal, toItem: aView, attribute: .centerX, multiplier: 1.0, constant: 0)
let constraintY = NSLayoutConstraint.init(item: cView, attribute: .centerY, relatedBy: .equal, toItem: aView, attribute: .centerY, multiplier: 1.0, constant: 0)
let constraintsWidth = NSLayoutConstraint.init(item: cView, attribute: .width, relatedBy: .equal, toItem: aView, attribute: .width, multiplier: 0.5, constant: 0)
let constraintsHeight = NSLayoutConstraint.init(item: cView, attribute: .height, relatedBy: .equal, toItem: aView, attribute: .height, multiplier: 0.5, constant: 0)
aView.addConstraints([constraintX,constraintY,constraintsWidth,constraintsHeight])

```




