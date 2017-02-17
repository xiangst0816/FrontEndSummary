# 浏览器兼容性问题及解决方案(CSS部分)

![可畏的前端](https://drscdn.500px.org/photo/72944089/m%3D2048/bee828b2bca1bca21d335bccae466489)

浏览器兼容性问题是指因为不同的浏览器对同一段CSS代码或者JS代码解析不同导致页面显示效果不统一或者脚本执行错误的情况。一般情况下，我们希望用户无论使用什么浏览器来查看网页效果都应该是一样的。浏览器的兼容性问题是Web前端开发人员经常会碰到的和必须要解决的问题。

浏览器兼容的部分一般分为CSS部分和JS部分，JS部分请参考[这里](http://www.jianshu.com/p/8cd605d14e19)。

相关的问题已近在前端界整理的很充分了，大致的问题可总结如下：

* 浏览器的初始化样式不同导致兼容性的问题
* 局部样式解析不同导致的bug

> 有些文章把样式使用技巧也放到兼容性的问题里面，这有失偏颇，关于常用样式使用技巧在[另一边文章](http://www.jianshu.com/p/25eaac282b0d)叙述，这里不重复。

## 1. 浏览器的初始化样式不同导致兼容性的问题

对于这个问题可以一上来就是用```*{margin:0;padding:0;}```，不过这个代码的杀伤力有些大，如果用户自定义样式未加载进来，则整个页面的布局毫无结构可言。

所以这里出现两个浏览器默认样式重置库：[Normalize.css](http://necolas.github.io/normalize.css/)和[reset.css](http://meyerweb.com/eric/tools/css/reset/)，上面的链接是项目的介绍。

关于这两个库的对比介绍也[比较详细](https://segmentfault.com/q/1010000000117189)，简单的说：**Normalize** 的理念则是尽量保留浏览器的默认样式，不进行太多的重置；而**Reset**的目的，是将所有的浏览器的自带样式重置掉，这样更易于保持各浏览器渲染的一致性。

浏览器默认样式重置就像是贴墙纸前先上一遍腻子将墙抹之后再贴。Normalize.css或者Reset.css需要在所有样式引用前引用。就我个人而言，使用Normalize.css比较多一些。

## 2. 局部样式解析不同导致的bug

其实，这里主要是**IE浏览器**和**非IE浏览器**之间的解析区别。如果使用了normalize或者reset后能处理大部分的问题，下面是剩余的部分：

#### 2.1 透明度的兼容CSS设置

**解决方式：** 

* IE：filter:progid:DXImageTransform.Microsoft.Alpha(style=0,opacity=60)。  
* 非IE：opacity:0.6。

#### 2.2 默认的盒子解析模型

标准 w3c 盒子模型的范围包括 margin、border、padding、content，并且 content 部分不包含其他部分； IE盒子模型的范围也包括 margin、border、padding、content。

和标准 w3c 盒子模型不同的是：ie 盒子模型的 content 部分包含了 border 和 pading。

因此，问题主要表现在css中的width是否计算border和padding的问题，这个是默认的盒子解析模型不同导致的。

**IE6：**中包括border和padding```(box-sizing: border-box;)```  
**IE7和非IE：**width宽度不包括border和padding```(box-sizing: content-box;)```  

**解决方式：**  根据使用方式，写好box-sizing属性。



#### 2.3 浮动问题

主要表现是子元素使用了浮动，而父元素没有撑开盒子，下面是解决的代码，加载父元素上就好。

**解决技巧：**

```css
.clearfix:before, .clearfix:after {   content: " "; /* 1 */   display: table; /* 2 */ }
 .clearfix:after {   clear: both; }
```


#### 2.4 高度不适应

高度不适应是当内层对象的高度发生变化时外层高度不能自动进行调节，特别是当内层对象使用margin 或padding时。

**例如：**

```html
#box {background-color:#eee; } 
#box p {margin-top: 20px;margin-bottom: 20px; text-align:center; } 
<div id="box">
<p>p对象中的内容＜/p>
</div>
```

**解决技巧：**

1. 在P对象上下各加2个空的div对象CSS代码{height:0px;overflow:hidden;}
2. 或者为DIV加上border属性

#### 2.5 滚动条颜色设置

IE无法设置滚动条颜色了, 解决办法是将body换成html 

**解决技巧：**

```css
html { 
	scrollbar-face-color:#f6f6f6; 
	scrollbar-highlight-color:#fff; 
	scrollbar-shadow-color:#eeeeee; 
	scrollbar-3dlight-color:#eeeeee; 
	scrollbar-arrow-color:#000; 
	scrollbar-track-color:#fff; 
	scrollbar-darkshadow-color:#fff; 
} 

```
#### 2.6 超链接访问过后hover样式就不出现的问题
被点击访问过的超链接样式不在具有hover和active了,很多人应该都遇到过这个问题,解决技巧是改变CSS属性的排列顺序: L-V-H-A 

**解决技巧：**

```css
a:link {} 
a:visited {} 
a:hover {} 
a:active {} 
```

#### 2.7 条件注释

在html中加入条件判断，选择激活哪些部分

**解决技巧：**

```html
<link rel="stylesheet" type="text/css" href="css.css" />

<!--[if IE 7]>
<link rel="stylesheet" type="text/css" href="ie7.css" />
<![endif]-->

<!--[if lte IE 6]>
<link rel="stylesheet" type="text/css" href="ie.css" />
<![endif]-->

lte -- 小于等于
lt  -- 小于
gte --  大于等于
gt  --  大于
！ --  不等于
```

#### 2.8 background-attachment:fixed在ios下失效的hack

ios系统和某些移动端background-attachment:fixed不兼容性，没有任何效果，但可以hack一下就可以了，代码如下:

ps：想在哪个标签加背景，可以在它class后:before.

```
body:before {
  content: ' ';
  position: fixed;
  z-index: -1;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background: url(path/to/image) center 0 no-repeat;
  background-size: cover;
}
```

#### 2.9 Calc使用

浏览器支持的不是很好，而且在使用的时候要加上厂商前缀，达到兼容性。另外，注意减号之间的空格。


```css
width: calc(100% - 80px);

#formbox {
  width:  130px;
  /*加厂商前缀，操作符（+，-，*，/）两边要有空格）*/               
  width:  -moz-calc(100% / 6);   
  width:  -webkit-calc(100% / 6);   
  width:  calc(100% / 6);   
  border: 1px solid black;
  padding: 4px;
}
```

## 最后

除去以上的总结，还有很多是关于在IE6下的Hack，但是这个“老古董”在市面上的占有率已近相当低了，另外，按照由上到下的开发模式(优先兼容高级浏览器)，IE8及以下的浏览器都可不用重点考虑。因此，关于样式兼容性的问题重点放到**浏览器样式重置**、**flex布局**、**CSS 预处理器(SCSS/Less)**、**CSS后处理器(Autoprefixer/Postcss/Cssnano)**上就可以解决大部分问题。

此外，关于Hack的问题不需要全部记住，只需要记住大概的问题点，在做布局的时候尽量避免，遇到问题查问题。我认为这部分问题会随着技术或者浏览器的更迭而全部解决，因此不需要放太多精力。


## 参考连接
* [Normalize.css 与传统的 CSS Reset 有哪些区别](https://www.zhihu.com/question/20094066)
* [关于CSS Reset 那些事（一）之 历史演变与Normalize.css](https://segmentfault.com/a/1190000003021766)
* [关于CSS Reset 那些事（二）之 Normalize.css 源码解读](https://segmentfault.com/a/1190000003025718)
* [关于CSS Reset 那些事（三）之 Normalize-zh.css 出炉](https://segmentfault.com/a/1190000003028985)