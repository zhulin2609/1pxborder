在视网膜屏（设备像素比为2）中实现1px border效果
=========

在设备像素比为2的屏幕上实现“1px”边框的效果

项目中中有在视网膜屏幕中实现1px border的需求，
首先，来看下面视觉给的输出图中的border：
 
从上面的视觉图可以看到，border是一根非常细的线。这篇文章将说明如何使用border-image实现在视网膜屏中1px的border效果。
注：因为硬件条件的限制，设备像素比（devicePixelRatio）为1的非视网膜屏手机无法达到这样的效果
首先准备一张符合你要求的border-image
通常手机端的页面设计稿都是放大一倍的，如：为适应iphone retina，设计稿会设计成640*960的分辨率，图片按照2倍大小切出来，在手机端看着就不会虚化，非常清晰。
同样，在使用border-image时，将border设计为物理1px，如下：
 
样式设置：

```
.border-image-1px ｛
  border-width: 0 0 1px 0;
  -webkit-border-image: url(linenew.png) 0 0 2 0 stretch;
  border-image: url(linenew.png) 0 0 2 0 stretch;
｝
```

我们这次是把border设置在边框的底部，所以使用的图片是2px高，上部的1px颜色为透明，下部的1px使用视觉规定的border的颜色。如果边框底部和顶部同时需要border，可以使用下面的border-image：
 
样式设置：
.border-image-1px {
  border-width: 1px 0;
  -webkit-border-image: url(linenew.png) 2 0 stretch;
  border-image: url(linenew.png) 2 0 stretch;
｝
到目前为止，我们已经能在iphone上展现1px border的效果了。但是我们发现这样的方法在非视网膜屏上会出现border显示不出来的现象，于是使用Media Query做了一些兼容，样式设置如下：
.border-image-1px {
  border-bottom: 1px solid #666;
｝

@media only screen and (-webkit-min-device-pixel-ratio: 2) {
    .border-image-1px {
        border-bottom: none;
        border-width: 0 0 1px 0;
        -webkit-border-image: url(../img/linenew.png) 0 0 2 0 stretch;
        border-image: url(../img/linenew.png) 0 0 2 0 stretch;
    }
}
参考文档：https://github.com/AlloyTeam/Mars/blob/master/solutions/border-1px.md
http://css-tricks.com/snippets/css/retina-display-media-query/
这是我们自己采用的方法，推荐大家使用。
下面介绍一下我们收集到的其他方法：
1、我们还尝试过@kevinffzeng 提供的一个方法，就是在html文件头部，将页面初始缩放比例设为0.5，即：<meta name="viewport" content="width=device-width, initial-scale=0.5,  user-scalable=no"/>
然后将css的样式都设为原始大小的两倍，但border-width仍设为1px。这种方法的确实现了在高分屏下呈现1px border的效果，但我们发现使用这种方法，在ios中可能产生字体大小异常的现象，至今还没查到原因。

2、使用background-image来实现。参考文档：https://excellenteasy.com/devblog/posts/how-to-target-physical-pixels-on-retina-screens-with-css/
这个方法是@yueqiao 提供的，我们使用过后，发现在一些安卓设备上，会出现border消失的情况。我们估计是低版本android上的浏览器不支持background-image属性。

3、高分屏下页面1px边框的解决方案——onepx.js：只需简单的敲上onepx(selectors); 即可实现全屏1px高清线。 文档：http://code.oa.com/v2/weima/detail/3933?comefrom=mail#comments
这个方法我们并没有尝试过，仅供参考。

4、还可以使用background-image。
跟border-image的方法一样，你要先准备一张符合你要求的图片： 
显然这里准备的是将border设置在底部
样式设置：
.background-image-1px {
  background: url(../img/line.png) repeat-x left bottom;
  -webkit-background-size: 100% 1px;
  background-size: 100% 1px;
｝
这是参考“街景探索”的做法，http://x.map.qq.com/wap/v/n/  推荐使用

