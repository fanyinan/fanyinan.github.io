---
layout: post
title: 易扩展的富文本控件

categories: [iOS]
description: 灵活、易配置、自带缓存和预加载的富文本控件，结合了策略模式和模板方法模式（尝试使用装饰者模式，却发现有点弄巧成拙）— 你值得拥有！！！
github: https://github.com/fanyinan/RichTextView-Swift-3.0
imagePrefix: /assets/source/2017-4-29-
---

公司在做即时通讯软件，所以显示文字消息是必须的。之前一直都在用label显示文字，像我一样单纯的文字，没有图文混排，没有点击事件，只有文字。但是偶然之间发现一个问题：在UITableView中显示文字的时候，由于cell的复用，当cell需要显示的时候才设置相应的文字，但是如果设置的文字过多（可多可多了）的时候，label在渲染的时候会有严重的性能问题，出现了特别明显的卡顿。很明显需要优化一下（通常没人会输入这么多字，但是这种已知的潜在的性能问题不解决一下怎么能忍），所以根据性能优化的经验，可以做一下异步渲染、预加载、缓存，能缓存和预加载的肯定就是处理成UIImage了，所以肯定需要CoreText。


<br />

### 简单介绍一下

---

先看一下大致结构：
<img src="{{page.imagePrefix}}1.png">

* 主要结构

	主要控件WZRichTextView继承自UIView，用于展示通过CoreText绘制文字生成的图片；通过WZTextStyle配置文字样式；可以配置多个Interpreter来处理文字
	

* 易扩展

	将对文字的处理的逻辑抽象出一个Interpreter，主要包括对文字的解析，点击时的样式，点击事件的处理，在上下文中进行绘制。通过具体的Interpreter来实现需要的接口完成相应的功能。传入的多个Interpreter按顺序依次处理文字，但是如果一个`CTRun`对应了多个Interpreter，只有第一个Interpreter的点击事件和点击时的样式会生效

* 灵活

	需要解析文字时只需传入相应的InterPreter即可，不做多余的文字处理。需要注意的是，当传入多个Interpreter时，需要注意在数组中的顺序，这个顺序决定了文字的处理顺序
		
* 缓存

	CoreText绘制文字生成的图片以及尺寸等相关信息会存入缓存中，以便高效的复用，缓存的key有文字内容、文字样式、Interpreter共同生成，防止发生混乱
	
* 预加载

	即使有缓存功能，但是在首次加载时依然会出现由于异步渲染无法立即显示文字的问题。预加载可以解决这个问题，文字尺寸的计算和绘制文字生成图片都是静态方式，不需要实例化WZRichTextView即可在控件加载之前便已完成对文字内容的缓存和高度的计算，当控件需要显示文字时便可直接在缓存中取得。所以使用者需要合理控制预加载的时机

<br/>

### 总会遇到的坑

---

1. 众所周知，当用`CoreText`进行图文混排的时候，可以将占位的标志替换成空格，再为	其添加`kCTRunDelegateAttributeName`样式。但是如果`CTLineBreakMode`的	值设为`byWordWrapping`，则会出现左图下面的情况，但实际上应该是右图的样子：

	<img src="{{page.imagePrefix}}2.png" width="45%" style="float: left; margin-right:5%">
	<img src="{{page.imagePrefix}}3.png" width="45%">

	这不是难点，赤裸裸的坑啊，也不懂是因为什么，这种乍一看像是苹果的bug的问题也不	太想深究，我的处理方法时把占位的标志替换成一个透明的字符而不是空格

2. `CTRun`指的是在一行中所有属性都相同的连续的文字，所以如果要显示三个连续的相同	的图片的话，那么绘制图片之前用到的这三个站位的字符的属性就必须加以区分。

	首先当做图文混排的时候，至少需要给这个占位的字符添加俩个属性，一个是自定义的属	性，存放图片的名字或其他什么可以索引到图片的方法；第二个就是	`kCTRunDelegateAttributeName`。如果第一个属性的值是一个字符串，第二个属	性的值是`CTRunDelegateCreate(&runDelegateCallbacks, nil)`，那么有一定几率（不知道为什么不是必现的）会在这本应该绘制多个图片的位置只绘制一个图片。
	
```swift

let insertSpace = NSMutableAttributedString(string: "*")

insertSpace.addAttribute(keyAttributeName, value: imageName, range: NSRange(location: 0, length: insertSpace.length))
let runDelegate = CTRunDelegateCreate(&runDelegateCallbacks, nil)
insertSpace.addAttribute(kCTRunDelegateAttributeName as String, value: runDelegate!, range: NSRange(location: 0, length: insertSpace.length))
```
		
并且这个时候在`runDelegateCallbacks`中也无法取得文字的高度来设置预留位置	的大小，通常是写死一个值，所以可以像下面这样写：
	
```swift
//控制图片尺寸的文字的相关尺寸
class PictureRunInfo {
  	var ascender: CGFloat
	var descender: CGFloat
	var width: CGFloat
  
	init(ascender: CGFloat, descender: CGFloat, width: CGFloat) {
 	self.ascender = ascender
 	self.descender = descender 
 	self.width = width
	}
}

//CTRunDelegateCreate中的回调
var runDelegateCallbacks = CTRunDelegateCallbacks(version: kCTRunDelegateVersion1, dealloc: { pointer in    
    }, getAscent: { pointer -> CGFloat in
      
      let pictureRunInfo = unsafeBitCast(pointer, to: PictureRunInfo.self)
      return pictureRunInfo.ascender
      
    }, getDescent: { pointer -> CGFloat in
      
      let pictureRunInfo = unsafeBitCast(pointer, to: PictureRunInfo.self)
      return pictureRunInfo.descender
      
    }, getWidth: { pointer -> CGFloat in
      
      let pictureRunInfo = unsafeBitCast(pointer, to: PictureRunInfo.self)
      return pictureRunInfo.width
    })


//成员变量，保存PictureRunInfo，防止被释放掉
var pictureRunInfos: [PictureRunInfo] = []


let insertSpace = NSMutableAttributedString(string: "*")
	
insertSpace.addAttribute(keyAttributeName, value: imageName, range: NSRange(location: 0, length: insertSpace.length))

//为注明的几个值只是几个控制位置的常量而已，不用太在意
let pictureRunInfo = PictureRunInfo(ascender: textStyle.font.ascender + extraHeight, descender: -textStyle.font.descender + extraHeight, width: imageSize.width + imageHoriMargin * 2)
pictureRunInfos += [pictureRunInfo]

let runDelegate = CTRunDelegateCreate(&runDelegateCallbacks, unsafeBitCast(pictureRunInfo, to: UnsafeMutableRawPointer.self))
insertSpace.addAttribute(kCTRunDelegateAttributeName as String, value: runDelegate!, range: NSRange(location: 0, length: insertSpace.length))
```

通过传一个`PictureRunInfo`的指针使不同的`CTRun`可以区分开，同时又可以准确的控制图片的位置

3. 现在还存在一个问题，就是无法明确Interpreter的顺序。虽然数组中的顺序即是处理文字的先后顺序，但是不够明确，比较理想的状态是可以和`GPUImage`一样使用装饰者来完成多个Interpreter之间的衔接，以次来强调处理文字的先后顺序。但是装饰者模式需要一个类来存储上一个元素，但是我不太希望通过派生出子类来自定义Interpreter，一是因为没有绝对的必要，不需要为了用一个设计模式而用设计模式，二是感觉用继承而不是`Protocol`会破坏其抽象性，所以最终还是放弃使用装饰者，之后有什么好的思路在加进来

PS：[RichTextView]({{page.github}})

不麻烦的话赏我个星星咯^_^

以上