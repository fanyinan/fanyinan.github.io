---
layout: post
title: 记此博客的诞生
<!-- description: original post at WOWMORON -->
<!-- tagline: original post at wowmoron.wordpress.com -->
categories: [技术]
tags: [Tech,生活]
---
奇迹般的迎来了我今年的第一个双休，周五白天还在火急火燎的改Bug，晚上就通知说明天休息吧，什么鬼。。。不过，正好得空建个博客，其实今年刚来北京那会就想建个博客用来积(zhuang)累(bi)了，只不过当时尚且年轻，没什么好写的，也就作罢。不过自从工作了这几个月的时间，或多或少也学到了点东西，谈不上什么经验分享，权当是个public笔记本吧 ^_^

先大体说一下这个博客的结构吧：

1.我使用Jekyll来创建博客，它来帮我们直接生成静态页面。对，没错，就是直接生成静态页面，没有服务器后台，没有数据库。 为什么用这个呢？因为我懒。我就想写点东西放到网上，我干嘛要去配置主机，干嘛要去整数据库，怎么省事怎么来吧 -_-#_  
2.其次使用了GitHub Pages来托管我的网站，Jekyll 可以运行在 GitHub Pages上，所以写完一篇文章之后你是需要push一下就OK了

先提一下，我用的MacBook(没钱买，公司的)，所以是在mac的环境下安装的，windows下估计差不多。先从Jekyll的搭建开始讲起，首先Jekyll是用Ruby写的，所以需要先搭建Ruby环境（写这篇文章的时候已经是下周二晚上了，我也不记得是不是需要安装Ruby环境，所以把Ruby环境干掉之后再启动Jekyll服务，提示找不到文件）

{% highlight ruby %}
# 先查看是否已经安装过ruby，安装过就算了
$ rvm list
#列出已知的ruby版本
$ rvm list known
#安装最新的
$ rvm install ruby-2.2.1
{% endhighlight %}

我一直翻着墙，没觉得慢，如果感觉死慢死慢的就用淘宝的镜像
{% highlight ruby %}
$ gem sources --remove https://rubygems.org/
$ gem sources -a https://ruby.taobao.org/
$ gem sources -l
    *** CURRENT SOURCES ***
    https://ruby.taobao.org
{% endhighlight %}

然后安装Jekyll
{% highlight ruby %}
$ gem install jekyll
{% endhighlight %}
如果提示安装Command-Line Tools，直接选Y安装就是了，<a href="http://jekyll.bootcss.com/docs/installation//">官方文档</a>上说会提示安装Xcode，因为我装好了，所以就不清楚咯

现在可以生成个Demo了
{% highlight ruby %}
#myblog就是博客文件夹的名字
$ jekyll new myblog
$ cd myblog
#博客主目录下启动服务
~/myblog $ jekyll serve
{% endhighlight %}

不报错就是起动成功了（废话），在浏览器打开http://localhost:4000 看是否真的成功了吧Y(^_^)Y_)

现在来把它搞到GitHub Pages上
首先需要有个GitHub的帐号，这个就不说了

然后建个仓库，再clone到本地，[官方文档](https://pages.github.com/)有详细教程，不过记得仓库名，username.github.io，username必须和owner的名字相同，否则只是个普通的仓库了

最后只要把用Jekyll创建的文件夹copy到本地仓库文件夹就OK了，只copy文件夹内的文件就行，根目录不需要，之后如果想在本地启动服务就可以直接进入本地仓库文件夹启动了。
现在来commit，sync，ok之后就可以在浏览器输入仓库名（也就是博客网址）来打开博客看看了

下面来整个域名，当然不用也可以，不过逼格就不那么的高了,推荐去godaddy上购买域名，毕竟是最大的域名提供商，不过我直接在万网上买了，购买的步骤就不写了，交钱拿货而已

说一下域名的解析设置吧

<img src="/assets/source/2015-12-2/1.png" width="100%">

记录类型一般是选`CANME`和`A(adderss)记录`：

`A记录`：将你的域名直接指向IP地址，这样DNS直接根据域名找到对应的IP

`CNAME`：简单来说就是给主机起一个别名，就像swift中的alias关键字，加个名字而已，本质上还是和原来的网址（也就是你的仓库名）做一样的事情，这样的一个好处是，如果IP地址换了，使用新注册的这个域名依然可以找到IP地址，不过A记录的方式就不可以了

不过我当时随便弄的，也没管太多，就用的A记录的

`主机记录`：用`www`做前缀就填`www`，不带就填`@`或不填，不过网上看到不带`www`的不能用`CNAME`，没有验证过，感兴趣的话可以试试

`解析线路`：默认就好

`记录值`：使用`CNAME`就填你的网址（仓库名）；使用`A记录`的话，用命令行`PING`一下你的网址（仓库名），把得到的IP地址填上就可以了

<img src="/assets/source/2015-12-2/2.png" width="100%">

保存，等个十几分钟就可以了

基本的建站方法就是这样，装修博客的工作就不多说了，对H5不熟悉的同学或向我一样比较懒的可以直接下载模板，有能力的话还是自己来写比较好

不管是下载模板还是自己写，都需要用到下面两个插件
{% highlight ruby %}
#高亮插件
$ gem install pygments.rb
#分页插件
$ gem install jekyll-paginate
{% endhighlight %}

下面是几个文档链接：

[markdown](http://www.appinn.com/markdown/)：写博客时用到的标注，很方便的说

[pygments](http://pygments.org)：代码高亮

[liquid](https://github.com/shopify/liquid/wiki/liquid-for-designers)：H5 liquid模版语法

[jekyll](http://jekyll.bootcss.com/docs/)：jekyll官方文档