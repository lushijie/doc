# 移动端 rem 布局

## 一、基本概念

### 1.物理像素(physical pixel)

一个物理像素是显示器(手机屏幕)上最小的物理显示单元，在操作系统的调度下，每一个设备像素都有自己的颜色值和亮度值。

### 2.设备独立像素(density-independent pixel)

设备独立像素(也叫密度无关像素，逻辑像素)，可以认为是计算机坐标系统中得一个点，这个点代表一个可以由程序使用的虚拟像素(比如: css像素)，然后由相关系统转换为物理像素。

### 3.设备像素比(device pixel ratio，简称dpr)

设备像素比 = 物理像素 / 设备独立像素 // 在某一方向上，x方向或者y方向。

在javascript中，可以通过window.devicePixelRatio获取到当前设备的dpr。

在css中，可以通过-webkit-device-pixel-ratio，-webkit-min-device-pixel-ratio和 -webkit-max-device-pixel-ratio进行媒体查询，对不同dpr的设备，做一些样式适配(这里只针对webkit内核的浏览器和webview)。


### 4. dpr导致的问题
<img src="http://p3.qhimg.com/t01e19e375adb434df0.png"/>

在普通屏幕下，1个css像素 对应 1个物理像素(1:1)；在retina 屏幕下，1个css像素对应 4个物理像素(1:4)。

理论上，1个位图像素对应于1个物理像素，图片才能得到完美清晰的展示。在普通屏幕下是没有问题的，但是在retina屏幕下就会出现位图像素点不够，从而导致图片模糊的情况。

<img src="https://p0.ssl.qhimg.com/t014caebc4d22336122.png" />

如上图：对于dpr=2的retina屏幕而言，1个位图像素对应于4个物理像素，由于单个位图像素不可以再进一步分割，所以只能就近取色，从而导致图片模糊(注意上述的几个颜色值)。

另一个问题，如果普通屏幕下，也用了两倍图片，会怎样呢？

很明显，在普通屏幕下，200×300(css pixel)img标签，所对应的物理像素个数就是200×300个，而两倍图片的位图像素个数则是200×300*4，所以就出现一个物理像素点对应4个位图像素点，所以它的取色也只能通过一定的算法(显示结果就是一张只有原图像素总数四分之一，我们称这个过程叫做downsampling)，肉眼看上去虽然图片不会模糊，但是会觉得图片缺少一些锐利度，或者是有点色差(但还是可以接受的)。

<img src="https://p0.ssl.qhimg.com/t011ea5ec62293b3cea.png" />

### 5. retina 高清问题

两倍图片(@2x)，然后图片容器缩小50%，如：图片大小，400×600;

1.img标签

```
  width: 200px;
  height: 300px;
```
  
2.背景图片

```
  width: 200px;
  height: 300px;
  background-image: url(image@2x.jpg);
  background-size: 200px 300px; // 或者: background-size: contain;
```
  
这样的缺点，很明显，普通屏幕下：

同样下载了@2x的图片，造成资源浪费。
图片由于downsampling，会失去了一些锐利度(或是色差)。
所以最好的解决办法是：不同的dpr下，加载不同的尺寸的图片。

### 6. retina border:1px 问题

<img src="https://p4.ssl.qhimg.com/t01aeac884aca9f9aac.png" />

iphone3gs(dpr=1)和iphone5(dpr=2)下面的测试效果，对比来看，对于1px的border的展示，它们是一致的

对于一条1px宽的直线，它们在屏幕上的物理尺寸(灰色区域)的确是相同的，不同的其实是屏幕上最小的物理显示单元，即物理像素，所以对于一条直线，iphone5它能显示的最小宽度其实是图中的红线圈出来的灰色区域，用css来表示，理论上说是0.5px。

所以，设计师想要的retina下border: 1px;，其实就是1物理像素宽，对于css而言，可以认为是border: 0.5px;，这是retina下(dpr=2)下能显示的最小单位。

然而，无奈并不是所有手机浏览器都能识别border: 0.5px;，ios7以下，android等其他系统里，0.5px会被当成为0px处理，那么如何实现这0.5px呢？

```
.scale{
  position: relative;
}
.scale:after{
  content:"";
  position: absolute;
  bottom:0px;
  left:0px;
  right:0px;
  border-bottom:1px solid #ddd;
  -webkit-transform:scaleY(.5);
  -webkit-transform-origin:0 0;
}
```

我们照常写border-bottom: 1px solid #ddd;，然后通过transform: scaleY(.5)缩小0.5倍来达到0.5px的效果，但是这样hack实在是不够通用(如：圆角等)，写起来也麻烦。


当然还有其他好多hack方法，网上都可以搜索到，但是各有利弊，这里比较推荐的还是页面scale的方案，是比较通用的，几乎满足所有场景。

对于iphone5(dpr=2)，添加如下的meta标签，设置viewport(scale 0.5)：

```
<meta name="viewport" content="width=640,initial-scale=0.5,maximum-scale=0.5, minimum-scale=0.5,user-scalable=no">
```

这样，页面中的所有的border: 1px都将缩小0.5，从而达到border: 0.5px;的效果。

### 7. 代码

```
  // add <meta charset="utf-8" name="viewport"> in html
  (function (doc, win) {
  function recalc () {

  // rem-unit plugin sublime
  var designWidth = 750; // 视觉图 750px, deviceWidth * dpr
  var standard = 100; // 基准 1rem = 100px
  var dpr = win.devicePixelRatio || 1;
  var scale = 1 / dpr;
  var docEl = doc.documentElement;

  // 设置data-dpr属性，留作的css hack之用
  docEl.setAttribute('data-dpr', dpr);

  // var deviceWidth = docEl.clientWidth;
  var deviceWidth = win.screen.width;
  if (win.orientation !== undefined) {
    // 横竖屏切换
    deviceWidth = win.screen[win.orientation === 0 ? 'width' : 'height'];
  }
  if (!deviceWidth) return;

  // 设置基准，设定最大值
  var rem = Math.min(deviceWidth * dpr * standard / designWidth, standard);

  // 设置viewport，进行缩放，达到高清效果
  var metaEl = doc.querySelector('meta[name="viewport"]');
  metaEl.setAttribute('content', 'width=' + dpr * deviceWidth +
    ',initial-scale=' + scale +
    ',maximum-scale=' + scale +
    ',minimum-scale=' + scale +
    ',user-scalable=no'
  );

  // 动态写入样式
  var styleEl = doc.createElement('style');
  docEl.firstElementChild.appendChild(styleEl);
  styleEl.innerHTML = 'html{font-size:' + rem + 'px!important;}'
  + 'body {font-size: ' + 30 + 'px!important;}';

  // 绑定关键参数到 window 对象
  win.deviceWidth = deviceWidth;
  win.dpr = dpr;
  win.rem = rem;

  // 给js调用的，某一dpr下rem和px之间的转换函数
  win.rem2px = function(v) {
    v = parseFloat(v);
    return v * rem;
  };
  win.px2rem = function(v) {
    v = parseFloat(v);
    return v / rem;
  };
  }

  // addEventListener
  if (!doc.addEventListener) return;
  var resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize';
  win.addEventListener(resizeEvt, recalc, false);
  doc.addEventListener('DOMContentLoaded', recalc, false);
  })(document, window);
    
```
