img延迟加载插件介绍
===

>我这是用的是这个插件**jquery.lazyload.js**,[关于介绍也很多了](http://blog.csdn.net/zzqw199012/article/details/18707473),我这里就只见上干货，就说怎么用开始，不过还是先介绍下他是干什么的，怎么干的。

##1. 做什么的
**图片延迟加载**，等**滚到图片的位置时才加载**，后面的图片先不加载，故能**避免卡顿**，需要加载图片的时候，图片进行下载，等加载完毕后才出现，**消除出现半幅图**的情况（改善用户体验）
##2. 怎么干的

	<img class="lazy" data-original="img/example.jpg" width="640" height="480">
	
- **data-original**:这个是真正要交加载的图片；
- **src**：这个是占位图片，默认是grey.gif，高度只有1px；自动添加的。你不管。
- **height**：**这个并不是必须设定**，如果没设定导致图片都堆到一起被看到，则启动加载（我测试过，没设定尺寸），so，你知道该怎么做，把图片包起来隔开。


引入jquery：

	<script src="jquery.js"></script>
	<script src="jquery.lazyload.js"></script>
	
在main.js中写上：

	$("img.lazy").lazyload();


这个例子我在[HTML5-WebAPP-template](https://github.com/xiangsongtao/HTML5-WebAPP-template)这里用到过了，你可以尝试下。

**如果使用的requirejs的话，这样做：**


```
/*如果将包通过bower引入的话,会自动在paths中添加的 */
require.config({
  paths: {
    bootstrap: '../../bower_components/bootstrap/dist/js/bootstrap',
    domReady: 'vendor/domReady.min',
    jquery: '../../bower_components/jquery/dist/jquery',
    'jquery.scrollstop': '../../bower_components/jquery_lazyload/jquery.scrollstop',
    'jquery.lazyload': '../../bower_components/jquery_lazyload/jquery.lazyload'
  },
  shim: {
    'jquery.lazyload': [
      'jquery'
    ],
    'jquery.scrollstop': [
      'jquery'
    ]
  }
});  
```

```
//先加载jquery插件及require和domReady模块
require(['require','jquery','domReady','jquery.lazyload','jquery.scrollstop'], function(require,$,domReady) {
  //之后通过require模块加载其余部分,当然这也是懒加载的方法
  require(['app','test'],function(app,test){
    domReady(function(){
      app();
      $("img.lazy").lazyload();
    })
  });
});
```

##3. 说下其他功能


###3.1设置加载阈值
>之前是“看到图片就加载”，现在是快到图片多少“**阈值**”的时候加载-提前一个距离加载。懂了不？

	$("img.lazy").lazyload({
	//还有200px就出现在屏幕上了，现在加载
    	threshold : 200
	});

###3.2使用jquery的事件触发img加载
>之前是自动，现在**根据事件触发加载**，例如click，mouseover。。。。。自定义事件也行。

	$("img.lazy").lazyload({
   	 	event : "click"
	});
	
自定义事件触发：sport时间5s后触发img加载

	$(function() {
    	$("img.lazy").lazyload({
        	event : "sporty"
    	});
	});

	$(window).bind("load", function() {
    	var timeout = setTimeout(function() {
        	$("img.lazy").trigger("sporty")
    	}, 5000);
	});

###3.3 使用img显现效果
>img下载好了后，用什么方式显现的问题，简单。应该使用的是jquery的动画，默认是用show(),估估计能替换成别的动画库。

	$("img.lazy").lazyload({
    	effect : "fadeIn"
	});


###3.4 如果img在一个scroll容器中（图片在带滚动条的容器里）

	#container {
    	height: 600px;
    	overflow: scroll;
	}

	$("img.lazy").lazyload({
     	container: $("#container")
	});

###3.5 如果img不是呈现队列的形式布局的
>这句话的意思是，我们默认：图片的出现顺序是与dom的结构顺序是一致的。可是有些时髦的布局会将当前该出现的img放到很远的地方，所以这个插件为了保证这个图片能够显示出来，就会在后面去找（如果找不到这个图片就在页显示不出来了），看能不能找到这个图片，就是这个值“failure_limit”，如果你的布局非常奇葩，那么，将这个值设为 "所有使用lazy功能的图片的总和"，这样，这个奇葩的图片就跑不掉了！呵呵。理解了不？

	$("img.lazy").lazyload({
	//向后找10个单位img
    	failure_limit : 10
	});

###3.6 如果图片不可见怎么办
> 跳过不可见的图片（Invisible）

	$("img.lazy").lazyload({
    	skip_invisible : true
	});
