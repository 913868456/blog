---
title:  AutoLayout Cell高度自适应 
date:  2018-11-22 16:27
categories:
- iOS
tags: 
- AutoLayout 
---

1. 不要设置TableView返回Cell高度的代理方法
2. 设置底部控件约束与cell的contentView距离
3. 设置预估行高,并且设置cell高度自动计算 

**注意:** 如果不设置预估行高的话,Cell会设置固定行高为44,如果使用SnapKit设置的Cell自适应高度可能在某些机型上无效

```
taskList.estimatedRowHeight = 77;
taskList.rowHeight = UITableViewAutomaticDimension;


```

有些时候,虽然设置了上面的动态约束,但是对cell高度做动态约束的时候,会在log里面有约束冲突的警告,这个时候是只需要将超过44的高度约束的Prority属性设置为大于750就行了,默认的高度约束是750;而默认高度的约束也是44,所以会出现冲突;
```
[LayoutConstraints] Unable to simultaneously satisfy constraints.
	Probably at least one of the constraints in the following list is one you don't want. 
	Try this: 
		(1) look at each constraint and try to figure out which you don't expect; 
		(2) find the code that added the unwanted constraint or constraints and fix it. 
(
    "<SnapKit.LayoutConstraint:0x2833606c0@MalfunctionDetailCell.swift#88 UILabel:0x103983790.top == UITableViewCellContentView:0x103984060.top + 10.0>",
    "<SnapKit.LayoutConstraint:0x283360a20@MalfunctionDetailCell.swift#95 UILabel:0x103983d70.top == UILabel:0x103983790.bottom + 6.0>",
    "<SnapKit.LayoutConstraint:0x283360d80@MalfunctionDetailCell.swift#101 UILabel:0x103983a80.top == UILabel:0x103983d70.bottom + 6.0>",
    "<SnapKit.LayoutConstraint:0x283360ea0@MalfunctionDetailCell.swift#104 UILabel:0x103983a80.height == 16.0>",
    "<SnapKit.LayoutConstraint:0x283360f00@MalfunctionDetailCell.swift#105 UILabel:0x103983a80.bottom == UITableViewCellContentView:0x103984060.bottom - 8.0>",
    "<NSLayoutConstraint:0x28346b8e0 'UIView-Encapsulated-Layout-Height' UITableViewCellContentView:0x103984060.height == 44   (active)>"  //这里是重点
)

```

#### Tips:  约束的优先级(Priority)
在Xib或者storyBoard 中我们设置的约束默认的优先级都是Required(1000),如果想要设置约束根据优先级动态调整,则需要将优先级降至1000以下;当然,我们也可以设置优先级的具体数值;
```
##注意## 
当我们将某个约束优先级设置为Required时,其他等级的约束将无效;
```
###### xib 和 storyBoard下有以下几种优先级
```
Required(1000) > High(750) > Low(250);
```
###### SnapKit增加一种
```
Required(1000) > High(750) > Medium(500) > Low(250);
```
