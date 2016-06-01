高性能JavaScript实践总结
=====================

本总结是对《高性能JavaScript》这本书的总结也是记录笔记,加深我对JavaScript的认识及实践技巧。

## 一、脚本的加载和执行

一般来说，JavaScript代码的执行会阻塞浏览器进行的其他程序，比如用户界面绘制。每次遇到```<script>```后，,页面都必须停下来等待代码下载并执行，然后再继续解析和渲染页面。在这期间，页面渲染和用户交互是完全被阻塞的，例如页面出现长时间的白屏。解决方案如下：  

1. 将```<script>```放在```</body>```之前，确保脚被执行前页面已完成渲染。
2. ```<script>```标签越少越好，可以考虑用gulp任务合并。因为HTTP请求会带来额外的性能开销。
3. 内嵌脚本如果在```<link>```之后会导致页面阻塞去等待样式表的下载。这样能获得最精准的样式信息，但是会阻塞其他任务。（不建议，但hack除外）
3. 使用无阻塞下载的方式
	* **延迟脚本：**   
		在```<script>```标签中使用defer属性，该属性指明此脚本不会修改DOM，因此可以安全的延迟执行。（async属性也用于异步加载脚本，区别在于其加载完成后自动执行，而defer需要页面完成后才执行）
	* **使用XHR对象下载JavaScript代码并注入到页面中**  
		主要的局限是只能在同域中请求js文件，也不能从CDN获取js文件。
	* **动态脚本（推荐）：**  
		 动态创建```<script>```元素并下载执行。即在window对象的load事件出发后再下载脚本。  
		 可自己实现(如下)，也可使用类库:[lazyload](http://github.com/rgrove/lazyload/)、[LABjs](http://labjs.com)。
		 
```
/**
 * @title: addTags动态加载js标签
 * @params:
 * tagUrl: js资源的数组
 * eachLoadedCB: 每个资源加载完毕的回调
 * allLoadedCB: 所有资源加载完毕的回调
 * */
function addTags(tagUrl, eachLoadedCB, allLoadedCB) {

    if (!(tagUrl instanceof Array)) {
        alert("first arguments must be array of urls");
        return false;
    }
    var totalResource = tagUrl.length;
    var restResource = totalResource;
    var downLoadPercent;
    for (var i = 0, len = tagUrl.length; len > i; i++) {
        //标签对象
        var _TagObjs;
        if (/.js$/.test(tagUrl[i])) {
            _TagObjs = document.createElement("script");
            _TagObjs.setAttribute('type', 'text/javascript');
            _TagObjs.setAttribute('src', tagUrl[i]);
            head.appendChild(_TagObjs);
        }
        if(_TagObjs.readyState){
            //IE
            _TagObjs.onreadystatechange = function () {
                if(_TagObjs.readyState == "loaded" || _TagObjs.readyState == "complete"){
                    _TagObjs.onreadystatechange = null;
                    restResource--;
                    downLoadPercent = Math.round((totalResource - restResource) / totalResource * 100);
                    !!eachLoadedCB && eachLoadedCB(totalResource, restResource, downLoadPercent);
                    !!!restResource && allLoadedCB && allLoadedCB();
                }
            }
        }else{
            _TagObjs.onload = function () {
                restResource--;
                downLoadPercent = Math.round((totalResource - restResource) / totalResource * 100);
                !!eachLoadedCB && eachLoadedCB(totalResource, restResource, downLoadPercent);
                !!!restResource && allLoadedCB && allLoadedCB();
            };
        }

    }
}


/**
 * 使用
 * */
addTags(["core.js","lib.js"],function (t,r,d) {
    alert("total:"+t+",rest:"+r+",percent:"+d);
},function () {
    alert("all loaded")
})
```	 		 		 
		 
## 二、数据存储

JavaScript的四种基本的数据存取位置，比较简单，这里只是简单列举：

- 字面量 （最快）
- 本地变量 （var创建）（最快）
- 数组元素 （最慢）
- 对象成员（最慢）

### 作用域链

找了一张图：
![作用域链](http://img.my.csdn.net/uploads/201304/04/1365082234_1223.png)

- 函数执行都会创建自己的执行环境，且执行环境都有自己的作用域链，用于解析标识符。函数运行时的变量对象会放在作用域链的顶端。当执行环境被销毁，此对象也随之销毁。故，在函数执行过程中，每遇到变量，都会经历一次标识符解析过程以决定从哪儿获取存储数据。---> **藏的越深，找的越慢！**

- 作用域链的末尾是全局变量对象，故**搜索该变量的过程必须遍历整个作用域链！**一般是讲此全局变量缓存的到局部变量中。

```
function init(){
	var a = document;
	var b = a.body;
	b.getElementByTagName("a");
}

```

- 改变作用域链的方式有：
	- with(不建议使用，可使用局部变量替换)
	- try catch (可用)  
	try内函数的作用域对象放在首位，catch内函数的作用域对象放在第二位。catch执行完毕，作用域链就返回到之前的状态。**建议将catch内的处理逻辑由专门的函数处理，由于只有一条执行语句，且没有局部变量的访问，作用域链的临时改变不会影响代码性能。**  
	
```
try{
	method();
} catch(e){
	handleErr(e);
}
```

### 闭包、作用域和内存

- 闭包会造成更多的内存开销
- 闭包会放在作用域链的顶端，第二层是活动对象的作用域，也就是说，闭包的存在会频繁的出现跨作用域访问标识符的情况，每次访问都会带来性能损失，解救方案：将常用的跨作用域的变量存储到局部变量中，然后直接访问局部变量。

### 原型链

- Javascript中的对象是基于原型的。
- 所有的对象都是Object的实例。
- 实例继承了原型链中的所有成员，故原型决定了实例的类型！

浏览器中的原型链结构:
![浏览器中的原型链结构](http://images.cnitblog.com/blog/543993/201401/211448230160.jpg)

- 嵌套的对象成员会显著的影响性能，访问的速度也越慢,例如：  
	window.location.href.toString() > window.location.href > location.href
- 缓存对象的成员，如果多次访问的话。  
```	var a = location;
	a.href;
	....
```	
- 属性和方法越深，访问的速度也越慢

## 三、DOM编程

- 减少DOM的访问次数，把运算留在JS端处理。
- 遍历数组的速度快于遍历集合是速度，故将HTML集合转化为数组在进行处理会更高效。


```
function toArray(coll){
	for(var i = 0,a = [],len = coll.length;i<len;i++){
		a[i] = coll[i];
	}
	return a;
}

var coll = document.getElementsByTagName("div");
var ar = toArray(coll);

```
- 建议使用querySlectorAll()方法


### 重绘和重排

- 重排：reflow，重新计算元素几何属性，重新构造渲染树，代价很高；
- 重绘： repaint，代价相对较小。

故：

- 批量修改DOM
- 批量改变样式（ele.style.cssText = ""）
- 使元素脱离文档流，避免大部分文档重排：
	- 动画元素绝对定位，脱离文档流	

### 事件委托

大量元素绑定相同处理事件的时候，将事件绑定到document上，通过冒泡事件捕获并处理

## 四、算法和流程控制

- for, while, do- while，性能相当，灵活使用
- 避免使用for-in，除非遍历对象的数量不确定。
- switch和if-else灵活使用
- 出现栈溢出，可能是因为使用了递归，将递归改为迭代可以避免。因为运行一个循环比反复调用一个函数的开销要少的多。

## 五、字符串和正则表达式

- 使用+和+=操作字符串，避免不必要的中间字符串
- 正则，待续

## 六、快速响应用户界面
- js任务不可超过100ms
- 使用settimeout将长时间的js放入队列中异步执行，不阻碍ui进程。
- 可以使用web works


## 七、AJAX
- 减少请求次数

## 八、编程实践
- 避免双重求值（使用eval()和Function()运行字符串代码）
- Object和Array使用直接量，而不是new
- 环境判断的函数应该使用延迟加载技术（Lazy Loading）或者条件预加载技术（Conditional Advance Loading），即，环境确定后，用该环境下的函数替换主函数

```
//延迟加载技术（Lazy Loading）
//第一次执行判断环境，之后修改addHandler函数并执行
functin addHandler(x){
	if(isThisEnvironment){
		addHandler = function(x){}
	}else{
		addHandler = function(x){}
	}
	addHandler(x);
}
```

```
//条件预加载技术（Conditional Advance Loading）
//脚本加载期间检测，并修改addHandler函数；
var addHandler = isThisEnvironment?function(x){}:function(x){};

```

- 使用js原生提供的方法（Math）
- 使用位操作

## 九、构建高性能js应用

- 合并多个JavaScript文件，减少HTTP请求数  
- **非必须的脚本延迟加载**
- JavaScript压缩，方法：ugify、js重构（GCC）
- 启用gzip
- manifest缓存技术，包含manifest的html也会被缓存
- CDN
- 文件名增加MD5值防止缓存更新问题

## 10、工具
- 分类：性能分析+网络分析
- 浏览器的开发工具
- Fidder/postman/Charles Proxy