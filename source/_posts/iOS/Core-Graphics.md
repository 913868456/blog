---
title:  CoreGraphics 
date:  2019-03-14 15:48
categories:
- iOS
tags: 
- CoreGraphics 
---
采用 `Quartz` 高级绘画引擎提供基于路径的绘制,形变,颜色管理,离屏渲染,渐变与阴影,图片数据管理,图片创建,图片蒙版,同样的PDF创建,显示和解析.

###几何数据类型
[CGFloat](https://developer.apple.com/documentation/coregraphics/cgfloat)

[CGPoint](https://developer.apple.com/documentation/coregraphics/cgpoint)

[CGSize](https://developer.apple.com/documentation/coregraphics/cgsize)

[CGRect](https://developer.apple.com/documentation/coregraphics/cgrect)

[CGVector](https://developer.apple.com/documentation/coregraphics/cgvector)

[CGAffineTransform](https://developer.apple.com/documentation/coregraphics/cgaffinetransform)

### 2D 绘制

[CGContext]()

#### 常用API说明

```

/*构建绘制路径*/
//在图形环境中创建一个新的空路径
func beginPath()
//在指定点开始一个新的子路径
func move(to: CGPoint)
//从当前点添加一条直线到指定点
func addLine(to: CGPoint)
//根据数组内的点连线,其中数组里的第一个点作为线的初始起点
func addLines(between: [CGPoint])
//添加一个矩形路径
func addRect(CGRect)
//添加多个矩形路径
func addRects([CGRect])
//添加椭圆
func addEllipse(in: CGRect)
//添加圆
func addArc(center: CGPoint, radius: CGFloat, startAngle: CGFloat, endAngle: CGFloat, clockwise: Bool)
//三阶贝塞尔曲线
func addCurve(to end: CGPoint, control1: CGPoint, control2: CGPoint)
//二阶贝塞尔曲线
func addQuadCurve(to end: CGPoint, control: CGPoint)
//关闭当前的子路径
func closePath()


/*绘制当前路径*/
//沿着当前路径绘制线
func strokePath()

/*绘制形状*/
//透明
func clear(CGRect)
//使用填充色在当前的图形状态下填充指定区域
func fill(CGRect)
//填充多个区域
func fill([CGRect])
//填充区域内的椭圆形
func fillEllipse(in: CGRect)
//画指定区域的路径
func stoke(CGRect)
//画椭圆
func stokeEllipse(in: CGRect)
//画多条线段
func stokeLineSegments(between: [CGPoint])

/*画渐变和阴影*/
//直线平滑过渡
func drawLinearGradient(_ gradient: CGGradient, 
                  start startPoint: CGPoint, 
                    end endPoint: CGPoint, 
                options: CGGradientDrawingOptions)

//扇面渐变过渡
func drawRadialGradient(_ gradient: CGGradient, 
            startCenter: CGPoint, 
            startRadius: CGFloat, 
              endCenter: CGPoint, 
              endRadius: CGFloat, 
                options: CGGradientDrawingOptions)               
            
/*绘制文字*/
//文字绘制位置
var textPosition: CGPoint
//设置字符间距
func setCharacterSpacing(CGFloat)
//设置字体
func setFont(CGFont)
func setFontSize(CGFloat)
func setTextDrawingMode(CGTextDrawingMode)

/*设置填充,描画,和阴影颜色*/
//填充色
func setFillColor(CGColor)
//阴影
func setShadow(offset: CGSize, blur: CGFloat)  
//描画线颜色
func setStokeColor(CGColor)  
//设置透明度
func setAlpha(CGFloat)


/*裁切路径*/   
//裁切区域内指定矩形
func clip(to: CGRect)
//裁切区域内多个矩形
func clip(to: [CGRect]) 
//裁切区域内用图片填充
func clip(to: CGRect, mask: CGImage)


/*设置路径绘制选项*/
//是否允许反锯齿
func setAllowAntialiasing(Bool)
//平滑度
func setFlatness(CGFloat)
//线帽
func setLineCap(CGLineCap)
//虚线
func setLineDash(phase: CGFloat, lengths: [CGFloat])
//线连接处样式
func setLineJoin(CGLineJoin)
//线宽
func setLineWidth(CGFloat)
 
/*保存上下文状态*/
func saveGState()
//保存最新的状态,需要之前保存过一次状态,否则会崩溃
func restoreGState()


/*管理图形上下文*/
//强制立即绘制
func flush()
//标记窗口上下文需要更新
func synchronize()

/*管理位图*/
//位图信息
var bitmapInfo: CGBitmapInfo
//像素高
var height: Int
//像素宽
var width: Int
//生成一个CGImage
func makeImage() -> CGImage?
               
```

[CGPath](https://developer.apple.com/documentation/coregraphics/cgpath)

[CGMutablePath](https://developer.apple.com/documentation/coregraphics/cgmutablepath)

[CGLayer]()

## 参考资料
[Core Graphics](https://developer.apple.com/documentation/coregraphics)
