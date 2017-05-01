---
layout: post
title: 浅谈UIImageOrientation（一）
description: UIImageOrientation －－－ 一个不起眼的属性
categories: [iOS]
tags: [Tech,Swift]
imagePrefix: /assets/source/2015-12-29- 
github: https://github.com/fanyinan/ImageOrientationDemo
---

我在之前做一个拼图游戏的时候，需要将一张图片裁成九份，于是我用`CGImageCreateWithImageInRect`，当我直接Assets中取图片是，everything is okay，然而当我在手机中真的选图片时，我就呵呵啦，理论上应该是图1中的样子，结果却是图2中的样子

<img src="{{page.imagePrefix}}1.png" width="47%" style="float: left; margin-right:3%">
<img src="{{page.imagePrefix}}2.png" width="50%">

很明显图片被逆时针旋转了90度，进过一番调查后发现是UIImaage的imageOrientation属性的原因

{% highlight swift %}
enum UIImageOrientation : Int {
    case Up
    case Down
    case Left
    case Right
    case UpMirrored
    case DownMirrored
    case LeftMirrored
    case RightMirrored
}
{% endhighlight %}

下图是得到这几个方向的图片的拍摄方法，原谅我没找到Mirrored的是怎么拍出来的，反正不是前置摄像头，(￣▽￣)

<img src="{{page.imagePrefix}}5.png" width="23%" style="float: left; margin-right:2%">
<img src="{{page.imagePrefix}}6.png" width="23%" style="float: left; margin-right:2%">
<img src="{{page.imagePrefix}}7.png" width="23%" style="float: left; margin-right:2%">
<img src="{{page.imagePrefix}}8.png" width="25%">

那这个属性有什么用呢？不知道大家有没有发现，在手机的相册中，无论拍摄照片时手机是什么方向，照片永远是拍摄时你在相机中看的的方向。为什么呢？我也不知道（不要打我），不过我推测是是`imageview`会帮你把图片旋转为.Up，不过图片本身并没有任何变化，只是看上去正了。

与此同时，`image`的`size`也就为校正之后的也就是把`image`直接放进`imageview`中是看到的`size`。也就是说，如果把一个方向为`.Right`的图片设为`.Up`，那么它的size也会改变(`width`和`height`互换)，因为图片已经是`.Up`了，不需要调整。

那么出现这种情况的原因为：

我将选择的图片在原图(方向为右)的情况下，裁成九份，假如这九张小图的`imageOrientation`依然为`.Right`，那么`imageview`也会把这九张小图也旋转为`.Up`,如下图：

<img src="{{page.imagePrefix}}3.png" width="50%">

很乱对不对，其实只要仔细看就会发现是把图2中的每张图顺时针旋转了90度。那么图2的情况是怎么来的呢，因为我在初始话裁剪后图片的时候，没有指定`imageOrientation`，则默认为`.Up`，`imageview`把`.Up`的图片旋转了0度，也就是没旋转，就成了图2的样子

{% highlight swift %}

//如果设置orientation为原图的imageOrientation，则为图3
UIImage(CGImage: CGImage, scale: CGFloat, orientation: UIImageOrientation)

//如果设置orientation为.Up(默认为.Up)，则为图2
UIImage(CGImage: CGImage)

{% endhighlight %}

不知道你有没有发现，我说这张图片的方向是`.Right`，可它明明是向左转了啊。为什么会这样呢？我也不知道。不过我猜...

+ 推测1：手机在Up的位置向右旋转90度拍出来的照片为.Right，同理向左旋转90拍出来的照片就为.Left

+ 推测2：当前的坐标系是左上角为原点，图片向右转就是逆时针旋转

<img src="{{page.imagePrefix}}4.png" width="50%">

###问题的解决

让我们来把图片转一转，然后用`contentsRect`是否可以取到正确的区域来检测是否真的旋转了，`contentsRect`会先确定要显示的范围，然后再调整图片，可以达到和直接裁剪图片一样的效果，至于`contentsRect`的具体用途就不做解释了

{% highlight swift %}

func fixOrientation(image: UIImage) -> UIImage {

    let imageRef = image.CGImage
    let width = image.size.width
    let height = image.size.height
    
    //创建一个位图上下文，具体用法在这不是关键
    let ctx = CGBitmapContextCreate(nil, Int(width), Int(height),
      CGImageGetBitsPerComponent(imageRef), 0,
      CGImageGetColorSpace(imageRef),
      CGImageGetBitmapInfo(imageRef).rawValue)
    
    //这个方法的作用是在这个上下文中所有绘制出来的东西都按照这个transform来变换，也可以看作是直接变换坐标系。
    //关键的getFixTransform方法在下面讲
    CGContextConcatCTM(ctx, getFixTransform(image))
    
    switch (image.imageOrientation) {
    case .Left, .LeftMirrored, .Right, .RightMirrored:

    //最后来绘制image，如果是Left或 Right的当然要取旋转90度之后的width和height
      CGContextDrawImage(ctx, CGRectMake(0, 0, height, width), imageRef)
    default:
      CGContextDrawImage(ctx, CGRectMake(0, 0, width, height), imageRef)
    }
    
    let fixImageRef = CGBitmapContextCreateImage(ctx)

    //这里记得不要将原图片的imageOrientation再设回来，否则imageview又会自动调整方向了
    let fixImage = UIImage(CGImage: fixImageRef!)
    return fixImage
    
  }

{% endhighlight %}

{% highlight swift %}

func getFixTransform(image: UIImage) -> CGAffineTransform {

    var transform = CGAffineTransformIdentity
    let width = image.size.width
    let height = image.size.height
    
    //调整图片的位置和方向
    switch (image.imageOrientation) {
    case .Down, .DownMirrored:
      transform = CGAffineTransformTranslate(transform, width, height)
      transform = CGAffineTransformRotate(transform, CGFloat(M_PI))
    case .Left, .LeftMirrored:
      transform = CGAffineTransformTranslate(transform, width, 0)
      transform = CGAffineTransformRotate(transform, CGFloat(M_PI_2))
    case .Right, .RightMirrored:
      transform = CGAffineTransformTranslate(transform, 0, height)
      transform = CGAffineTransformRotate(transform, CGFloat(-M_PI_2))
    default: // .Up, .UpMirrored:
      break
    }
    
    //处理Mirrored的情况
    switch (image.imageOrientation) {
    case .UpMirrored, .DownMirrored:
      transform = CGAffineTransformTranslate(transform, width, 0)
      transform = CGAffineTransformScale(transform, -1, 1)
    case .LeftMirrored, .RightMirrored:
      transform = CGAffineTransformTranslate(transform, height, 0)
      transform = CGAffineTransformScale(transform, -1, 1)
    default: // .Up, .Down, .Left, .Right
      break
    }
    
    return transform
  }

{% endhighlight %}

代码不难看懂，只是不够具象，下面我以一个.Right的图片来举个例子

{% highlight swift %}

transform = CGAffineTransformTranslate(transform, 0, height)
transform = CGAffineTransformRotate(transform, CGFloat(-M_PI_2))

{% endhighlight %}  

__注意:坐标系原点在左下角__

<img src="{{page.imagePrefix}}9.png" width="95%">

  
__因为`CGContextConcatCTM`相当于改变的整个坐标系，所以一定要先平移，再旋转，否则会出现下面的情况__

  
<img src="{{page.imagePrefix}}10.png" width="95%">

其他几种情况包括Mirrored变换的都大同小异。

嗯，就到这了，最后附上演示Demo源码地址[GitHub]({{page.github}})
