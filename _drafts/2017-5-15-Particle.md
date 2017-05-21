---
layout: post
title: 仿Particle.js
description: 一款仿Particle.js的动画特效控件
categories: [iOS]
tags: [Tech, Swift]
imagePrefix: /assets/source/2017-5-15/
github: https://github.com/fanyinan/Particle
---

在某网站无意中看到这个效果，感觉不错，就想自己试着实现一下：
<img src="{{page.imagePrefix}}1.png" width="100%">

在网页源码中找到了[Partice](https://github.com/VincentGarreau/particles.js)，我对JS并不是很熟悉，但是在勉强读了一下源码之后对这个控件有了一个大体的了解。点和线的绘制使用canvas来绘制的，也就是`Core Graphics`。所以很明显会想到他如果用`WebGL`的话，会有更