---
title: 项目总结
date: 2017-05-06 20:38:38
categories:
  - 项目总结
---
# ikea移动端项目总结
差不多花了大半个月和同学同事一起完成了ikea项目的一期。虽然不是第一次做移动端前端开发，但是以前做的大都是微信公众号的开发，这次是要兼容两者；以前一般为图省事会直接引入meat标签固定页面宽度为750px，这次尝试使用rem的兼容模式。

## 移动设备兼容

>考虑到这一点的时候，情不自禁去百度了一下，刚好看到一篇很[合适的博客](http://www.codeceo.com/article/font-size-web-design.html)，于是心中立马有了思路，既然rem是以html的font-size为基准，那么如果整张网页都是以rem为单位，在不同的屏幕下就只是相当于按屏幕大小的比例缩放了，效果应该还挺不错，至少是ui同事最希望看到的，于是立刻上手。

>在这里借鉴了上段提到的博客中网易的做法。使用rem布局结合在html上根据不同分辨率设置不同font-size有很多不好解决的麻烦，网易的解决发发是页面上html的font-size不是预先通过媒介查询在css里定义好的，而是通过js计算出来的，所以当分辨率发生变化时，html的font-size就会变，不过这得在你调整分辨率后，刷新页面才能看得到效果。计算规则的话，就和ui图的尺寸有关了，拿我自己的来说，我拿到的ui图的横向分辨率是750px，为了计算方便，取一个100px的font-size为参照，那么body元素的宽度就可以设置为width: 7.5rem，于是html的font-size=deviceWidth / 7.5，这样如果标注题上给的width是100px，写css时直接写1rem就可以了，所以在js代码中加了如下：

```

    document.documentElement.style.fontSize = document.documentElement.clientWidth / 7.5 + 'px';

```

>其实早知道事情不会这么一帆风顺，由于平时习惯性的会将js代码放在html末尾引入，所以果然出了幺蛾子，在html的dom结构加载完成时才加载了这句js，然后这句js改变了html的font-size的大小，所以这就不可避免的导致了页面上每个元素的大小都发生了改变，也就是说页面刷新出来会看到一次很明显的回流，简直是一种差到爆的用户体验。苦思冥想了十分钟，暂时想到一种不是很完美的解决办法，在css中添加了一段媒体查询，对主流的几种手机屏幕先判断一下并且设定html的font-size，由于css在head部位引入，所以可以一定程度上解决问题，下面是媒体查询代码：


```
@media screen and (min-width:0px) and (max-width:321px){html{font-size:42.67px}}
@media screen and (min-width:322px) and (max-width:376px){html{font-size:50px}}
@media screen and (min-width:377px) and (max-width:415px){html{font-size:55.2px}}
@media screen and (min-width:416px) and (max-width:501px){html{font-size:66.7px}}
@media screen and (min-width:502px) and (max-width:639px){html{font-size:85.2px}}
@media screen and (min-width:640px) and (max-width:719px){html{font-size:95.8px}}
```

>说这种方法时一定程度上解决主要是因为查询的边界是按照几款iphone的屏幕尺寸定的，如果遇到不在这几种边界周围的安卓手机，应该还是会有回流现象发生，接下来还要再想一些完美的解决方案。

## 微信和普通浏览器的兼容

>做这个兼容主要是在某些情况能使用微信js-sdk的api方法。举个例子，在微信微信浏览器中上传图片时，如果不使用微信提供的图片上传方法，那么安卓手机只能选择拍照（无法从相册中选择），所以这里先判断了是否是微信浏览器，如果是则使用微信提供的api，如果不是则老老实实input上传。下面是判断的方法：

```
function is_weixn(){  
    var ua = navigator.userAgent.toLowerCase();  
    if(ua.match(/MicroMessenger/i)=="micromessenger") {  
        return true;  
    } else {  
        return false;  
    }  
}
```


## 新的尝试

>1.之前写项目总是习惯用jquery，jquery也确实提供了丰富的dom操作方法，但一位师兄提醒我，如果使用原生的js来书写dom操作，会让你对dom树的结构，节点的选取、操作有更深刻的理解。所以这全程使用的原生的js，个人感觉虽然略显冗余，但是在这之后做牛客网上关于dom操作的题确实顺手了好多，也发现了自己对原生js的很多方法使用的还是不够熟练。

>2.以前遇到一些稍复杂一点的功能总是喜欢找一些轻量级的插件来辅助实现，这次也算是时间充裕，而且自己也立了flag不引入任何插件和框架，所以就自己动手完成了一些功能。主要有两个，一个是轮播，一个是上拉加载；这两个小插件在[github](https://github.com/wangwei0724/plugs)上都有。

>>轮播主要要实现的是图片滑动和无限滚动。图片滑动是将需要轮播的图片横向放在一个div中，如果需要轮播n张图片，此div的宽度就为n+2张图片的宽度之和（比如有5张100px的图需要轮播，那么该div的宽度就是700px，理由在后面），给改div加上overflow：hidden的样式，让其多出来部分不显示出来；并且给div加上定位属性，然后通过调整div的left值改变图片位置以达到轮播效果；无限滚动其实是在第一张图前面额外添加一张最后一张图，最后一张图后面额外添加第一张图，这也解释了为什么上文div的宽度要设置为n+2张图片的宽度之和。还是拿五张图来举个例子，假设五张图的代号分别为1、2、3、4、5，添加完图片后的排列就是5123451，这样图片滑动到5时，再向后就会滑到最后一张1，此时将div的left值设置为第一张1的left值，就可以完成瞬间拉回第一张图，视觉上不会察觉。

>>上拉加载的主要思路是检测onscroll判断height，如果scrollpHeight=clientHeigh+scrollTop就向后端发起ajax请求，也就是说计算滚动条位置加上网页可视高度，如果等于页面高度时触发加载函数。