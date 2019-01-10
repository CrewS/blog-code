---
title: 回顾css3相关知识
date: 2018-10-23 14:54:04
tags:
    - css3
    - 动画效果
---

## css3中的变形（transform）、过渡(transtion)、动画(animation)

### 一、transform
用法：transform: none|transform-functions;  
transform-functions：rotate | scale | skew | translate |matrix;  
```css
#div1{
  width: 50px;
  height: 50px;
  background:#de0010;
  transform: rotate(45deg) scale(0.5);
}
```
效果是旋转45读，缩小50%
##### rotate
ratate(angle) 定义 2D 旋转，在参数中规定角度。正直顺时针，负值逆时针
##### scale
```css
scale(<number>[, <number>])：提供执行[sx,sy]缩放矢量的两个参数指定一个2D scale（2D缩放）
```
scaleX()只对x轴方向进行矢量缩放，scaleY()则对Y轴
基点为元素中心点，也可以通过transform-origin来改变元素基点位置
##### skew
扭曲变形
与scale一样同样具有三种情况：skew(x,y)使元素在水平和垂直方向同时扭曲（X轴和Y轴同时按一定的角度值进行扭曲变形）；skewX(x)仅使元素在水平方向扭曲变形（X轴扭曲变形）；skewY(y)仅使元素在垂直方向扭曲变形（Y轴扭曲变形）
##### tanslate
移动translate我们分为三种情况：translate(x,y)水平方向和垂直方向同时移动（也就是X轴和Y轴同时移动）；translateX(x)仅水平方向移动（X轴移动）；translateY(Y)仅垂直方向移动（Y轴移动）

##### matrix
```css
matrix(<number>, <number>, <number>, <number>, <number>, <number>)
```
 matrix以一个含六值的(a,b,c,d,e,f)变换矩阵的形式指定一个2D变换，相当于直接应用一个[a b c d e f]变换矩阵。就是基于水平方向（X轴）和垂直方向（Y轴）重新定位元素
##### 改变元素基点transform-origin
transform-origin(X,Y)或者transform-origin: x;改变元素基点


### 二、transition
transition主要包含四个属性值：执行变换的属性：transition-property,变换延续的时间：transition-duration,在延续时间段，变换的速率变化transition-timing-function,变换延迟时间transition-delay。下面分别来看这四个属性值
##### transition-property
transition-property是用来指定当元素其中一个属性改变时执行transition效果，其主要有以下几个值：none(没有属性改变)；all（所有属性改变）这个也是其默认值；indent（元素属性名）。当其值为none时，transition马上停止执行，当指定为all时，则元素产生任何属性值变化时都将执行transition效果，ident是可以指定元素的某一个属性值。
##### transition-duration
transition-duration是用来指定元素 转换过程的持续时间，取值：<time>为数值，单位为s（秒）或者ms(毫秒),可以作用于所有元素，包括:before和:after伪元素。其默认值是0，也就是变换时是即时的。
##### transition-timing-function
transition-timing-function的值允许你根据时间的推进去改变属性值的变换速率，transition-timing-function有6个可能值:
- ease：（逐渐变慢）,默认值，ease函数等同于贝塞尔曲线(0.25, 0.1, 0.25, 1.0).
- linear：（匀速),linear 函数等同于贝塞尔曲线(0.0, 0.0, 1.0, 1.0).
- ease-in：(加速),ease-in 函数等同于贝塞尔曲线(0.42, 0, 1.0, 1.0).
- ease-out：（减速），ease-out 函数等同于贝塞尔曲线(0, 0, 0.58, 1.0).
- ease-in-out：（加速然后减速），ease-in-out 函数等同于贝塞尔曲线(0.42, 0, 0.58, 1.0)
- cubic-bezier：（该值允许你去自定义一个时间曲线),特定的cubic-bezier曲线。(x1, y1, x2, y2)四个值特定于曲线上点P1和点P2。所有值需在[0, 1]区域内，否则无效。

##### transition-delay
transition-delay是用来指定一个动画开始执行的时间，也就是说当改变元素属性值后多长时间开始执行transition效果，其取值：<time>为数值，单位为s（秒）或者ms(毫秒)，其使用和transition-duration极其相似，也可以作用于所有元素，包括:before和:after伪元素。 默认大小是"0"，也就是变换立即执行，没有延迟。

### 三、animation
css3中animation与html的canvasb不一样，animation只应用在已有的元素上
在w3school里 animation属性
一、animation-name:是用来定义一个动画的名称，其主要有两个值：IDENT是由Keyframes创建的动画名，换句话说此处的IDENT要和Keyframes中的IDENT一致，如果不一致,将不能实现任何动画效果；none为默认值，当值为none时，将没有任何动画效果。另外我们这个属性跟前面所讲的transition一样，我们可以同时附几个animation给一个元素，我们只需要用逗号“，”隔开。
二、animation-duration：是用来指定元素播放动画所持续的时间长，默认值为0
三、animation-timing-function:它和transition中的transition-timing-function一样，6个取值
四、animation-delay:是用来指定元素动画开始时间
五、animation-iteration-count:用来指定元素播放动画的循环次数，默认值为1，infinite为无限次数循环
六、animation-direction: 动画播放方向
七、animation-play-state:主要是用来控制元素动画的播放状态。其主要有两个值，running和paused其中running为默认值。

css3的animation是由Keyframes（帧）实现效果的，那么keyframes怎么定义呢
```css
@keyframes IDENT {
    from {
        Properties:Properties value;
    }
    Percentage {
        Properties:Properties value;
    }
    to {
        Properties:Properties value;
    }
}
```
IDENT是一个动画名称，percentage是过程的百分比，Properties是属性，value是属性值
一个简单的例子,红色的方块一直在旋转
```css
#div1{
  width: 100px;
  height: 100px;
  background:#de0010;
  -webkit-animation:mymove infinite;
  -webkit-animation-duration:2s;
}
@-webkit-keyframes mymove{
    0% {
        transform: rotate(0); 
    }
    25% {
        transform: rotate(45deg); 
    }
    50% {
        transform: rotate(180deg); 
    }
    75% {
        transform: rotate(225deg); 
    }
    100% {
        transform: rotate(360deg); 
    }
}
```


