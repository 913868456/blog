
---
title:  CALayer及CAAnimation 
date:  2018-04-27 15:39
categories:
- iOS
tags: 
- CoreAnimation 
---
## Layer
- Layer可以绘制的动画
![动画类型](http://upload-images.jianshu.io/upload_images/3340896-901e4849bf35fa8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- CALayer position 和 anchorpoint

![positon属性](http://upload-images.jianshu.io/upload_images/3340896-64a48bd8dd7ba3f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![锚点属性](http://upload-images.jianshu.io/upload_images/3340896-38913e949dcbfeac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

       - anchorpoint 基于单位坐标系,默认 (0.5,0.5)
       - position    基于点坐标系,具体位置由锚点决定
    frame 属性是在 positon和bounds属性下衍生出来的,frame相对于父类坐标系,bounds定义自身坐标系

![锚点位置与postion的变化](http://upload-images.jianshu.io/upload_images/3340896-d58622e2018f19a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![锚点如何影响图层变化](http://upload-images.jianshu.io/upload_images/3340896-1dde00133bac2ba9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![图层变化矩阵数据配置](http://upload-images.jianshu.io/upload_images/3340896-54966cd3653e1120.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- Layer Tree 分类

    - 模型树(Model Tree)
    - 显示树(Presentation Tree)
    - 渲染树(Render Tree)

- 图层(CALayer) 添加动画(CAAnimation)
动画基于操作图层对象来展示效果,通过下列方法来给图层添加动画
```
- (void)addAnimation:(CAAnimation *)anim 
              forKey:(NSString *)key;
```

## CAAnimation
- CoreAnimation层级
![层级](http://upload-images.jianshu.io/upload_images/3340896-f0db3c26d671677e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- CoreAnimation如何绘制内容
![绘制流程](http://upload-images.jianshu.io/upload_images/3340896-ac2ef3c4d239d1aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- CoreAnimation API

```
 CAAnimationGroup `animations`
 CAPropertyAnimation `keypath`
      - CABasicAnimation `fromValue` `toValue`
      - CAKeyframeAnimation `values` `path` `keyTimes`
 CATransition `type` `subType` `startProgress` `endProgress`
```

- 关于动画时间的控制

动画暂停

```
//根据全局时间获取本地时间
CFTimeInterval pausedTime = [layer convertTime:CACurrentMediaTime() fromLayer:nil];
//将图层动画速度设为0
layer.speed = 0.0;
//根据暂停的本地时间设置偏移
layer.timerOffset = pausedTime;
```
动画恢复

```
//得到偏移时间
CFTimeInterval pausedTime = layer.timeOffset;

//恢复图层的速度
layer.speed = 1.0;

//设置timerOffset
layer.timeOffset = 0.0;

//设值为0,进行时间转换
layer.beginTime = 0.0;

//beginTime在当前时间的左侧
CFTimeInterval timeSincePause = [layer convertTime:CACurrentMediaTime() fromLayer:nil] - puasedTime;
layer.beginTime = timeSincePause;

```
```
//动画恢复的时间点,要求:
1.beginTime = currentTime - timeOffset
2.timeOffset = 0
```

## CAMediaTiming Delegate
CAAimation, CALayer 都遵循 CAMediatiming 协议,都包含下列属性 
`begintTime` `speed` `timeoffset` `duration` `repeatDuration` `repeatCount` `fillmode`

![CAMediaTiming的理解](https://upload-images.jianshu.io/upload_images/3340896-2a7edb149c839e71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 参考资料

[Core Animation Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40004514)

[xiaolinyeyi](https://blog.csdn.net/xiaolinyeyi/article/details/51736907)

[DH's Den](https://blog.csdn.net/u013282174/article/details/51605403)
