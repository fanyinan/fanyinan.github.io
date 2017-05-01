---
layout: post
title: UIScrollView中的双击图片放大—zoomToRect
<!-- tagline: original post at wowmoron.wordpress.com -->
categories: [iOS]
tags: [Tech,Swift]
github1: https://github.com/fanyinan/WZPhotoBrowser
github2: https://github.com/fanyinan/ZoomImageScrollViewDemo
imagePrefix: /assets/source/2016-8-5-
---

突然有一天，我想给我的[图片浏览器]({{page.github1}})加上双击放大功能，想想这么简单又遍地都是的功能应该很好加吧。然而并没有，我被打脸了。

虽然找到了这个方法`zoomToRect(_ rect: CGRect, animated animated: Bool)`，但是完全不懂怎么用，第一个参数`rect`到底该传个什么东西。官方解释是：

`A rectangle defining an area of the content view. The rectangle should be in the coordinate space of the view returned by viewForZoomingInScrollView:`

`rect`是要显示的内容的区域，需要在`viewForZoomingInScrollView:`的返回的view的坐标空间内

开始真的不懂什么意思（好吧，我傻），不过在做了几次测试后，知道了这货表示什么了。不过在说这个之前，我先简单说明一下我是怎么实现图片缩放的：

1. 新建一个继承UIScrollView的类，用作控制缩放的容器；
2. 新建一个UIImageView，`frame.size`为要显示图片的`image.size`；
3. 实现`UIScrollViewDelegate`的`viewForZoomingInScrollView:`方法，返回值为需要缩放的`UIImageView`。此时`contentSize`始终为这个`UIImageView`的`frame.size`，所以不需要特别设置`contentSize`
4. 根据长边计算把图片完全显示出来的缩放比例`zoomScale`，同时设置最小缩放比例`minimumZoomScale`和`zoomScale`相同，为了当图片被捏合缩小时可以反弹回来；然后随意设置最大缩放比例`maximumZoomScale`，我设置的为`3`
5. 最后，当`scrollView`调用`layoutSubviews`时，修改`UIImageView`的`frame.origin`，保证图片可以显示在中间

__第四步的代码__

```swift
 private func calculateZoomScale() {
    
    let boundSize = bounds.size
    let imageSize = image.size
    
    let scaleX = boundSize.width / imageSize.width
    let scaleY = boundSize.height / imageSize.height
    
    var minScale = min(scaleX, scaleY)
    
    //当图片的尺寸比scrollView小时，不进行缩放，显示原比例
    if scaleX > 1 && scaleY > 1 {
      minScale = 1
    }
    
    minimumZoomScale = minScale
    zoomScale = minimumZoomScale
    maximumZoomScale = 3

  }
```

__第五步的代码__

{% highlight swift %}

 override func layoutSubviews() {
    
    moveFrameToCenter()
    
  }

  private func moveFrameToCenter() {
    
    let boundsSize = bounds.size
    let imageViewSize = imageView.frame.size
    
    //1
    var adjustX: CGFloat = 0
    var adjustY: CGFloat = 0
    
    //2
    if boundsSize.width > imageViewSize.width {
      adjustX = (boundsSize.width - imageViewSize.width) / 2
    }
    
    //3
    if boundsSize.height > imageViewSize.height {
      adjustY = (boundsSize.height - imageViewSize.height) / 2
    }
    
    imageView.frame.origin = CGPoint(x: adjustX, y: adjustY)
    
  }
{% endhighlight %}

**第五步的代码说明**

1. 可以想象`scrollView`有一个`contentView`，`origin`为`contentOffset`，`size`为`contentSize`，`imageView`在这个`contentView`中，且与之重合，当滑动`scrollView`时，`imageView`的`frame.origin`不会改变，始终为0
2. 当`scrollView`的`width`比`imageView`的`width`大，就把`imageView`的水平位置设在`contentView`中间
3. 同2

<img src="{{page.imagePrefix}}1.png" width="23%" style="float: left; margin-right:2%">
<img src="{{page.imagePrefix}}2.png" width="23%" style="margin-right:2%">

绿色的部分可以认为是`contentView`的位置


### 下面进入正题

根据代码来解释

{% highlight swift %}
	
  //1
  func imageViewDoubleTap(tap: UITapGestureRecognizer) {
  
  	//2
    guard zoomScale == minimumZoomScale else {
      setZoomScale(minimumZoomScale, animated: true)
      return
    }
    
    //3
    let zoomRectScale: CGFloat = 2
    //4
    let position = tap.locationInView(imageView)
    
    //5
    let zoomWidth = frame.width / zoomScale / zoomRectScale
    let zoomHeight = frame.height / zoomScale / zoomRectScale
    //6
    let zoomX = position.x - zoomWidth / 2 - imageView.frame.minX / zoomScale / zoomRectScale
    let zoomY = position.y - zoomHeight / 2 - imageView.frame.minY / zoomScale / zoomRectScale

    //7
    zoomToRect(CGRect(x: zoomX, y: zoomY, width: zoomWidth, height: zoomHeight), animated: true)
    
  }

{% endhighlight %}

**代码说明**

1. 双击事件加在`imageView`上，只有双击`imageView`才进行缩放
2. 当`scrollView`的`zoomScale`不是最小比例时，设为最小比例，用于当放大后再次双击时缩小
3. 设置双击放的的比例，这里设置为双击放大2倍
4. 获取点击在`imageView`的位置，这里值得一提的是，虽然此时`imageView`被缩小，其`frame`也是当前被缩小时的值，但是`position`依然是未被缩小时的位置，比如：图片尺寸为`640*300`，那么`imageView`在未被缩小时的尺寸也是`640*300`，假设被缩小后为`320*150`，此时点击`imageView`的右下角，我们得到的`position`为`640*300`
5. 这一步是关键，计算放大的尺寸。先来解释一下`zoomToRect`的参数，这是一个`CGRect`的值，表示一块矩形区域，这个区域指的是当zoomScale为1时，取`contentView`的`CGRect`的部分，然后把`contentView`按长边的比例缩放(自动计算`zoomScale`，当然我们不用关心这个`zoomScale`是多少)，使这部分填充`scrollView`, 
先看图：
<img src="{{page.imagePrefix}}3.png" width="48%" style="float: left; margin-right:2%">
<img src="{{page.imagePrefix}}4.png" width="48%" style="margin-right:2%">
先来算宽度，我们把放大的比例设为2（这个比例为我们的变量`zoomRectScale`，而不是`zoomScale`），那么假如我们要把图片放大1倍（也就是不变），显示在`scrollView`中的是整张图片，所以当放大1倍时，这个宽度为图片的宽度；同理，当放大2倍时，显示在`scrollView`中的部分的宽度为1倍时的1/2，也就是图片宽度的一半，即`image.size.width / zoomRectScale`。
当然这里指的是当图片的尺寸比`scrollView`大的情况，但是当图片很小的时候，极端一点，一张`10*10`的图片，当放大时需要把长度为5（宽度的1/2）放大到屏幕宽，显然不是2倍。所以如下图，我们需要把蓝色框部分当做一张图片来进行放大，就得到`(image.size.width + imageView.frame.minX / zoomScale * 2) / zoomRectScale`，由于`image.size.width`等价于`imageView.frame.width / zoomScale`，所以也就是`frame.width / zoomScale / zoomRectScale`（`frame.width`为`scrollView`的宽度）。高度也是同理
<img src="{{page.imagePrefix}}5.png" width="70%">。
6. 再来计算`origin`，现在我们有了中点坐标，也就是`position`，也有了宽度，那么`zoomX`就很容易计算，`position.x - zoomWidth / 2`，`zoomY`同理。
但是这里有一个特殊情况，就是当一张图片其长宽都小于`scrollView`，但是放大后的长或宽大于`scrollView`长或宽的的时候，放大图片后，图片偏左。如下图（为了方便观察，我把scrollView的长和宽都缩小1/2，这样可以看到超出的部分。右图是双击图片中点放大后的样子）：                         
<img src="{{page.imagePrefix}}6.png" width="48%" style="float: left; margin-right:2%">
<img src="{{page.imagePrefix}}7.png" width="48%" style="margin-right:2%">
那么这是为什么呢，我也不知道，真的不知道，只是发现当执行`zoomScale`之后，`scrollView`的`contentOffset`会比预期的值增加了`imageView.frame.minX`，也就是说上方右图的`imageView`向左距中心偏移的值等于上方左图中`imageView`的`frame.minX`，但是我不知道为什么。
先来看这个问题怎么解决吧，既然`scrollView`的`contentOffset`增加了，使得图片向左移动了，就导致图片的左侧的一部分被遮住了，那么我们选取的放大区域就需要向左移动一点，也就是计算`zoomX`的时候可以减去一个值来调整位置。由于图片在放大后偏移了`imageView.frame.minX`，而且放大倍数为2，那么在未放大时需要向左调整的距离就为`imageView.frame.minX / zoomScale / zoomRectScale`，所以就需要`position.x - zoomWidth / 2 - imageView.frame.minX / zoomScale / zoomRectScale`。`zoomY`同理
7. 执行缩放

就酱


**Github链接**

[图片浏览器Demo]({{page.github1}})

[此文章Demo]({{page.github2}})
