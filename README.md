街景wap官网中有在视网膜屏幕中实现1px border的需求.
首先，来看下面视觉给的输出图中的border：
![](http://7tszky.com1.z0.glb.clouddn.com/Fg8OjQ3yHWrQyg0IA0kXG-GMn7uf)

从上面的视觉图可以看到，border是一根非常细的线。这篇文章将说明如何使用border-image实现在视网膜屏中1px的border效果。

> 注：因为硬件条件的限制，设备像素比（devicePixelRatio）为1的非视网膜屏手机无法达到这样的效果

首先准备一张符合你要求的border-image：

![](http://7tszky.com1.z0.glb.clouddn.com/FmfBKPpD4hdPyvQFjaCHAFBco9Jp)

通常手机端的页面设计稿都是放大一倍的，如：为适应iphone retina，设计稿会设计成640*960的分辨率，图片按照2倍大小切出来，在手机端看着就不会虚化，非常清晰。
同样，在使用border-image时，将border设计为物理1px，如下：



样式设置：
```css
.border-image-1px {
    border-width: 0 0 1px 0;
    -webkit-border-image: url(linenew.png) 0 0 2 0 stretch;
    border-image: url(linenew.png) 0 0 2 0 stretch;
}
```
上文是把border设置在边框的底部，所以使用的图片是2px高，上部的1px颜色为透明，下部的1px使用视觉规定的border的颜色。如果边框底部和顶部同时需要border，可以使用下面的border-image：

![](http://7tszky.com1.z0.glb.clouddn.com/FiSW72mbvaG8DYPJ8ae5CI9FQZxI)

样式设置：

```css
.border-image-1px {
    border-width: 1px 0;
    -webkit-border-image: url(linenew.png) 2 0 stretch;
    border-image: url(linenew.png) 2 0 stretch;
}
```
到目前为止，我们已经能在iphone上展现1px border的效果了。但是我们发现这样的方法在非视网膜屏上会出现border显示不出来的现象，于是使用Media Query做了一些兼容，样式设置如下：

```css
.border-image-1px {
    border-bottom: 1px solid #666;
} 

@media only screen and (-webkit-min-device-pixel-ratio: 2) {
    .border-image-1px {
        border-bottom: none;
        border-width: 0 0 1px 0;
        -webkit-border-image: url(../img/linenew.png) 0 0 2 0 stretch;
        border-image: url(../img/linenew.png) 0 0 2 0 stretch;
    }
}
```

>参考文档：
https://github.com/AlloyTeam/Mars/blob/master/solutions/border-1px.md
http://css-tricks.com/snippets/css/retina-display-media-query/

下面介绍一下其他方法：
- 设置viewport
直接按照设计师提供的640px宽的设计稿来重构，然后通过控制viewport的initial-scale值为0.5进行缩放，这种方案在ios下可以完美运行（淘宝就是这么做的），但是由于android下不支持initial-scale，所以这个方案不适用于android。

```html
<meta name="viewport" content="initial-scale=0.5,user-scalable=no"/>
```
- background-image
跟border-image的方法一样，你要先准备一张符合你要求的图片：

![](http://7tszky.com1.z0.glb.clouddn.com/FmfBKPpD4hdPyvQFjaCHAFBco9Jp)

此例是准备将border设置在底部
样式设置：

```css
.background-image-1px {
    background: url(../img/line.png) repeat-x left bottom;
    -webkit-background-size: 100% 1px;
    background-size: 100% 1px;
}
```
- box-shadow

```css
.box-shadow-1px {
    box-shadow: inset 0px -1px 1px -1px #c8c7cc;
}
```
使用box-shadow都会让线有阴影，甚至颜色变浅。但是使用box-shadow与使用border类似，代码量少，使用方便，而且可以设置圆角矩形，在精细度要求不高的情况下可以尝试使用这种方案。
- 渐变背景
与background-image方案类似，只是将图片替换为css3渐变。
样式设置：

```css
.background-gradient-1px{
   background: -webkit-gradient(linear, left top, left bottom, color-stop(.5, transparent), color-stop(.5, #c8c7cc), to(#c8c7cc)) left bottom repeat-x;
   background-size: 100% 1px;
}
```
该方案不能满足1px圆角矩形。
- 缩放
边框由一个元素来承载，将这个元素的高度（或宽度）设置为1px，然后再将该元素缩放1倍。
样式设置：

```css
.scale-1px{
   position: relative;
}
.scale-1px:after{
   content:"";
   position: absolute;
   bottom:0px;
   left:0px;
   right:0px;
   border-bottom:1px solid #c8c7cc;
   -webkit-transform:scaleY(.5);
   -webkit-transform-origin:0 0;
}
```
- 听说Firefox和Safari8已经支持`0.5px`的单位了，代码可以像下面这样写：

```css
div{
   border:1px solid black;
}

@media (-webkit-min-device-pixel-ratio: 2){
 div{
    border-width:0.5px;
 }
}
```
不过`0.5px`这个单位有点过于颠覆前端开发的认识了twitter上有位哥们已经被震惊的不知所云
![](http://7tszky.com1.z0.glb.clouddn.com/Fig9jfAAD1MzK8aZG50NxQHKag4o)

- 基于`border-image`和`transform`使用Sass的线下解决方案:
Mixin：sass中使用@mixin声明混合，可以传递参数，参数名以$符号开始，多个参数以逗号分开，也可以给参数设置默认值。声明的@mixin通过@include来调用。
1. 基于border-image：

_onepx.scss:

```sass
@mixin onepx($selector, $position: bottom, $color: #666, $onepxImgDirname: './img/linenew.png') {
  #{$selector} {
    border-#{$position}: 1px solid $color;
  }

  @media only screen and (-webkit-min-device-pixel-ratio:2) {
    #{$selector} {
      border-#{$position}: none;
      @if $position == 'bottom' {
        border-width: 0 0 1px 0;
        -webkit-border-image: url($onepxImgDirname) 0 0 2 0 stretch;
        border-image: url($onepxImgDirname) 0 0 2 0 stretch;
      } @else if $position == 'top' {
        border-width: 1px 0 0 0;
        -webkit-border-image: url($onepxImgDirname) 2 0 0 0 stretch;
        border-image: url($onepxImgDirname) 2 0 0 0 stretch;
      } @else if $position == 'right' {
        border-width: 0 1px 0 0;
        -webkit-border-image: url($onepxImgDirname) 0 2 0 0 stretch;
        border-image: url($onepxImgDirname) 0 2 0 0 stretch;
      } @else if $position == 'left' {
        border-width: 0 0 0 1px;
        -webkit-border-image: url($onepxImgDirname) 0 0 0 2 stretch;
        border-image: url($onepxImgDirname) 0 0 0 2 stretch;
      }  
    }
  }
}
```

test.scss:

```sass
@import "onepx";

.container {
  @include onepx('.myonepx', 'top', '#666', './img/linenew.png');
}

@include onepx('.border-top', 'top');
@include onepx('.border-bottom');
```

执行bat文件：
> sass --scss --style expanded test.scss test.css

生成css代码：

```css
.container .myonepx {
  border-top: 1px solid "#666";
}
@media only screen and (-webkit-min-device-pixel-ratio: 2) {
  .container .myonepx {
    border-top: none;
    border-width: 1px 0 0 0;
    -webkit-border-image: url("./img/linenew.png") 2 0 0 0 stretch;
    border-image: url("./img/linenew.png") 2 0 0 0 stretch;
  }
}

.border-top {
  border-top: 1px solid #666666;
}

@media only screen and (-webkit-min-device-pixel-ratio: 2) {
  .border-top {
    border-top: none;
    border-width: 1px 0 0 0;
    -webkit-border-image: url("./img/linenew.png") 2 0 0 0 stretch;
    border-image: url("./img/linenew.png") 2 0 0 0 stretch;
  }
}
.border-bottom {
  border-bottom: 1px solid #666666;
}

@media only screen and (-webkit-min-device-pixel-ratio: 2) {
  .border-bottom {
    border-bottom: none;
    border-width: 0 0 1px 0;
    -webkit-border-image: url("./img/linenew.png") 0 0 2 0 stretch;
    border-image: url("./img/linenew.png") 0 0 2 0 stretch;
  }
}
```
2. 基于transform的缩放：

_onpx.scss

```sass
@mixin _prefixDpr($selector, $position: 'top', $pseudo: 'before', $dpr: '2') {
  @media only screen and (-webkit-min-device-pixel-ratio:$dpr) {
    @if $dpr == '1.5' {
      #{$selector}:#{$pseudo} {
        -webkit-transform: scaleY(.7);
        transform: scaleY(.7);
        @if $position == 'top' {
          -webkit-transform-origin: left top;
        } @else if $position == 'bottom' {
          -webkit-transform-origin: left bottom;
        }
      }
    } @else if $dpr == '2' {
      #{$selector}:#{$pseudo} {
        -webkit-transform: scaleY(.5);
        transform: scaleY(.5);
        @if $position == 'top' {
          -webkit-transform-origin: left top;
        } @else if $position == 'bottom' {
          -webkit-transform-origin: left bottom;
        }
      }
    } @else if $dpr == '3' {
      #{$selector}:#{$pseudo} {
        -webkit-transform: scaleY(.3);
        transform: scaleY(.3);
        @if $position == 'top' {
          -webkit-transform-origin: left top;
        } @else if $position == 'bottom' {
          -webkit-transform-origin: left bottom;
        }
      }
    }
  }
}

@mixin onepx($selector, $position: 'top',$pseudo: 'before', $color: #666) {
    #{$selector}:#{$pseudo} {
      content: ' ';
      display: block;
      border-top: 1px solid $color;
      position: absolute;
      left: 0;
      right: 0;
    }
    #{$selector} {
        position: relative;
        &:#{$pseudo} {
          @if #{$position} == 'top'{
            top: 0;
          } @else if #{$position} == 'bottom' {
            bottom: 0;
          }
        }
    }
    @include _prefixDpr($selector, $position, $pseudo, '1.5');
    @include _prefixDpr($selector, $position, $pseudo, '2');
    @include _prefixDpr($selector, $position, $pseudo, '3');

}
```
test.scss

```sass
@import "onepx";

.container {
  @include onepx('.myonepx');
}

@include onepx('.hello', 'bottom', 'after', '#777');
```
执行bat文件
> sass --scss --style expanded test.scss test.css
生成css代码：

```css
.container .myonepx:before {
  content: ' ';
  display: block;
  border-top: 1px solid #666666;
  position: absolute;
  left: 0;
  right: 0;
}
.container .myonepx {
  position: relative;
}
.container .myonepx:before {
  top: 0;
}
@media only screen and (-webkit-min-device-pixel-ratio: 1.5) {
  .container .myonepx:before {
    -webkit-transform: scaleY(0.7);
    transform: scaleY(0.7);
    -webkit-transform-origin: left top;
  }
}
@media only screen and (-webkit-min-device-pixel-ratio: 2) {
  .container .myonepx:before {
    -webkit-transform: scaleY(0.5);
    transform: scaleY(0.5);
    -webkit-transform-origin: left top;
  }
}
@media only screen and (-webkit-min-device-pixel-ratio: 3) {
  .container .myonepx:before {
    -webkit-transform: scaleY(0.3);
    transform: scaleY(0.3);
    -webkit-transform-origin: left top;
  }
}

.hello:after {
  content: ' ';
  display: block;
  border-top: 1px solid "#777";
  position: absolute;
  left: 0;
  right: 0;
}

.hello {
  position: relative;
}
.hello:after {
  top: 0;
}

@media only screen and (-webkit-min-device-pixel-ratio: 1.5) {
  .hello:after {
    -webkit-transform: scaleY(0.7);
    transform: scaleY(0.7);
    -webkit-transform-origin: left bottom;
  }
}
@media only screen and (-webkit-min-device-pixel-ratio: 2) {
  .hello:after {
    -webkit-transform: scaleY(0.5);
    transform: scaleY(0.5);
    -webkit-transform-origin: left bottom;
  }
}
@media only screen and (-webkit-min-device-pixel-ratio: 3) {
  .hello:after {
    -webkit-transform: scaleY(0.3);
    transform: scaleY(0.3);
    -webkit-transform-origin: left bottom;
  }
}
```

好处：可以使用习惯的sass写1px的实现方案，并且支持传参，更加灵活。

> 参考：
http://www.topcss.org/?p=769
