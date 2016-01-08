---
layout: post
title: 浅谈UIImageOrientation（二）
description: UIImageOrientation －－－ 我是续集
categories: [技术]
tags: [Tech,Swift]
imagePrefix: /assets/source/2016-1-7- 
github: https://github.com/fanyinan/ImageOrientationDemo2
lastPage: http://fanyinan.me/浅谈UIImageOrientation(一)
---

为何总遇到关于图片的方向的问题，是不是我打开的方式不对  orz

在做一个裁剪图片的时候，发现裁剪得到的部分并不是选定的部分，然后有了之前的经验，就联想到时`UIImageOrientation`的问题

###来说一下问题的原因

直接上图

<img src="{{page.imagePrefix}}2.png" width="100%">

看着可能拥挤了点，不过应该能看清楚，如果有不明白的可以去看[浅谈UIImageOrientation（一）]({{page.lastPage}})

###解决它

转换`CGRect`的代码如下：

{% highlight swift %}

private func transformOrientationRect(originImage: UIImage, rect: CGRect) -> CGRect {
    
    var rectTransform: CGAffineTransform = CGAffineTransformIdentity
    
    switch originImage.imageOrientation {
    case .Left:
      rectTransform = CGAffineTransformMakeRotation(CGFloat(M_PI_2))
      rectTransform = CGAffineTransformTranslate(rectTransform, 0, -originImage.size.height)
    case .Right:
      rectTransform = CGAffineTransformMakeRotation(CGFloat(-M_PI_2))
      rectTransform = CGAffineTransformTranslate(rectTransform, -originImage.size.width, 0)
    case .Down:
      rectTransform = CGAffineTransformMakeRotation(CGFloat(-M_PI))
      rectTransform = CGAffineTransformTranslate(rectTransform, -originImage.size.width, -originImage.size.height)
    default:
      break
    }
    
    let orientationRect = CGRectApplyAffineTransform(rect, CGAffineTransformScale(rectTransform, originImage.scale, originImage.scale))
    
    return orientationRect
    
  }

{% endhighlight%}

接着上面的图说，依然以`.Right`为例：

{% highlight swift %}

rectTransform = CGAffineTransformMakeRotation(CGFloat(-M_PI_2))
rectTransform = CGAffineTransformTranslate(rectTransform, -originImage.size.width, 0)

{% endhighlight%}

这部分代码没什么需要解释的，只要知道CGRect是怎么转换的就好，下面的图片应该能看的很清楚：

<img src="{{page.imagePrefix}}3.png" width="48%" style="float: left; margin-right:2%">
<img src="{{page.imagePrefix}}4.gif" width="50%">

旋转的时候看起来是不是很奇怪，但隐约感觉有什么规律，那再看下面这个GIF，是不是秒懂

<img src="{{page.imagePrefix}}5.gif" width="70%">

UIView的CGAffineTransformTranslate实际上也是这样的

其他几个方向的图片也和`.Right`的一样，就不再赘述了

嗯，就到这了，最后附上演示Demo的源码地址[GitHub]({{page.github}})

