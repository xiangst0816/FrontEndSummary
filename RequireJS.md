 RequireJS 思路整理
===

>第一次看官网的说明真是一头雾水,再次学习的时候感觉思路清晰很多,因此在这总结.

为什么使用requirejs?
---
**下面是我认为的原因:**

1. 代码逻辑复杂需要模块化管理;
2. 在js文件中引入其他js文件的代码只能通过标签```<script>```
3. 提升网页打开速度需要减少调用外部的js标签```<script>```

接下来我就从怎么使用开始吧.

需要在html代码中添加requirejs的`<script>`标签
---
**两种方式都行:**  
```<script data-main="scripts/main.js" src="scripts/require.js"></script>```  
```<script data-main="scripts/main" src="scripts/require.js"></script>```

1. 将标签添加在`<body>`的末尾
1. 指定js文件的主入口位置`main.js`
2. 指定`require.js`文件位置

配置main.js文件
---


	**目录结构**
	www/
	----index.html
	----js/
	--------app/
	------------sub.js
	------------inventory.js
	------------cart
	--------lib/
	------------jquery.js
	------------canvas.js
	--------main.js
---
	requirejs.config({
    	//默认路径
    	baseUrl: 'js/lib',
    	
    	//在baseUrl默认路径的基础上,加载app.js文件,他现在
    	//在lib文件夹的上级目录中(与js目录同级),不能再app后
    	//加上js后缀!!!
    	//在baseUrl目录下可以不用在paths中写明.例如canvas
    	paths: {
        	main: '../main',
        	jquery: 'jquery.1.1.min',
        	sub: '../app/sub',
        	inventory.js: '../app/inventory.js',
        	cart: '../app/cart',
        	
    	}
	});

	// 主入口
	//前面是依赖的模块数组(也可以是模块的位置url), 后面是对应的**调用名字**(已经对位)
	requirejs(['jquery', 'canvas', 'app/sub'],
	function     ($,      canvas,     sub) {
    	//你的逻辑代码
    	//sub.color;
    	//sub(value);
    	});
    	
    	
定义模块sub.js
---

###1. 如果只含有键值对的模块
	**from sub.js**
	define(function () {
	
    	//做一些初始设置
		
		//return出去的就是sub模块提供的信息
		//此处为一个object
    	return {
        	color: "black",
        	size: "unisize"
    	}
	});
	
###2. 如果存在依赖的键值对的模块
	**from sub.js**
	//可requirejs一样,前面是*.js的path(前面已定义),后面是回调函数(传入
	//对应的变量名)
	define(["cart", "inventory"], function(cart, inventory) {
        //return模块
        return {
            color: "blue",
            size: "large",
            addToCart: function() {
                inventory.decrement(this);
                cart.add(this);
            }
        }
    }
	);
	
###3. 返回函数的模块
	**from sub.js**
	//和上面一样
	define(["cart", "inventory"], function(cart, inventory) {
        //return模块
        return function(color){
            color: "color",
            size: "large",
            addToCart: function() {
                inventory.decrement(this);
                cart.add(this);
            	}
            }
        }
    }
	);
	
###4. 包装CommonJS的代码

	**from sub.js**
	//
	define(function(require, exports, module) {
		//require需要的模块a和b,和上面一样的用法
        var a = require('a'),
            b = require('b');

        //Return CommonJS的结果
        return function () {};
    	}
	);
	
###5.不建议在模块内部对模块直接命名
	// "foo/title" 这样的命名不建议,
	define("foo/title",
        ["my/cart", "my/inventory"],
        function(cart, inventory) {
            //Define foo/title object in here.
       }
    );
    
    
    
其他注意事项
---
####1.一个文件一个模块
####2. 在define内部再次引用模块的情况
**需要将"require"本身作为一个依赖注入到模块中**

	define(["require", "./relative/name"], function(require) {
    	var mod = require("./relative/name");
	});
	
####3. 生成相对于模块的URL地址
**需要将"require"本身作为一个依赖注入到模块中**, 结果的url不需要加.js后缀   

	define(["require"], function(require) {
    	var cssUrl = require.toUrl("./style.css");
	});
	
	
循环依赖
---
循环依赖(a依赖b，b同时依赖a),则在这种情形下当b调用a模块时，它会得到一个undefined的a, 解决办法如下,将require作为依赖注入进来. 


	//Inside b.js:
	define(["require", "a"],
    	function(require, a) {
        	//在这种情形下当b调用a模块时，它会得到一个undefined的a,
        	return function(title) {
        		//调用a中的doSomething()函数
            	return require("a").doSomething();
        	}
    	}
	);

JSONP服务依赖-官网的翻译真烂
---

  仅支持返回值类型为JSON object的JSONP服务，其他返回类型如数组、字串、数字等都不能支持。  
  为了在RequireJS中使用JSON服务，需要指定一个callback参数,这里使用"defineFn"。这意味着你可将JSONP URL获取到的值看成是一个模块(返回的是一个函数defineFn(jsonValue))。
  	
  	require(["http://example.com/api/data.json?callback=defineFn"],
    	function (data) {
        	//这里的data就是返回的defineFn(jsonValue)
        	console.log(data);
    	}
	);

requirejs的机制
---

RequireJS使用head.appendChild()将每一个依赖加载为一个script标签。
RequireJS等待所有的依赖加载完毕，计算出模块定义函数正确调用顺序，然后依次调用它们。

requirejs.config的配置
---
- **baseUrl** ：所有模块的查找根路径。
- **paths** ：path映射那些不直接放置于baseUrl下的模块名。设置path时起始位置是相对于baseUrl的，除非该path设置以"/"开头或含有URL协议（如http:）。
- **shim**: 将shim定义模块设置为全局可用(类似于path的作用),设定该模块的依赖项,但不加载该依赖, 只是到用到shim定义模块时才加载依赖项, 类似于插件而存在

		requirejs.config({
			path: {
				jquery:"jquery",
				underscore:"underscore"
			},
			shim: {
        		'backbone': {
        		deps: ['underscore', 'jquery'],
        		exports: 'Backbone'
        	},
        	'underscore': {
            exports: '_'
        },
        	'foo': {
            	deps: ['bar'],
            	exports: 'Foo',
            	init: function (bar) {
        		return this.Foo.noConflict();
            	}
        	}
    	}
		});
        	
        define(['backbone'], function (Backbone) {
  			return Backbone.Model.extend({});
		});	
        	
 **Backbone强制依赖于 Underscore.js,非强制依赖于 jQuery/Zepto.故  **  
 
 - shim配置**仅设置了代码的依赖关系**，想要实际加载shim指定的或涉及的模块，仍然需要一个常规的require/define调用。设置shim本身不会触发代码的加载。 
 - 成为shim脚本依赖的三个可选条件:
 	- 使用其他"shim"模块作为shim脚本的依赖，
 	- 在调用define()之前定义了全局变量(如jQuery或lodash)的AMD库, 这样调用不会出错(例如将jquery先在path中声明)。
 	- CMD库  	

- **map**:对于给定的模块前缀，使用一个不同的模块ID来加载该模块。  
该手段对于某些大型项目很重要：如有两类模块需要使用不同版本的"foo"，但它们之间仍需要一定的协同。 在那些基于上下文的多版本实现中很难做到这一点。而且，paths配置仅用于为模块ID设置root paths，而不是为了将一个模块ID映射到另一个。     

		requirejs.config({
    		map: {
        		'some/newmodule': {
            		'foo': 'foo1.2'
        		},
        		'some/oldmodule': {
            		'foo': 'foo1.0'
        }
    		}
		});

当“some/newmodule”调用了“require('foo')”，它将获取到foo1.2.js文件；  
而当“some/oldmodule”调用“`require('foo')”时它将获取到foo1.0.js。

- **config**:**将配置信息向下传给一个模块**。这些配置往往是application级别的信息，需要一个手段将它们向下传递给模块。在RequireJS中，基于requirejs.config()的config配置项来实现。要获取这些信息的模块可以**加载特殊的依赖“module”**，并调用**module.config()**。

  		requirejs.config({
    		config: {
        		'bar': {
            		size: 'large'
        		},
        		'baz': {
            		color: 'blue'
        		}
    		}
		});
		//调用config的内容
  		define(['module'], function (module) {
    		//Will be the value 'blue'
    		var color = module.config().color;
		});
		//commonJS模式
		define(function (require, exports, module) {
    		//Will be the value 'large'
    		var color = module.config().size;
		});
  	
 版本支持, requirejs延迟加载, 依赖注入!!!
 ---
 >**require函数**的变态用法,直接上代码, 记住require(或者requirejs, 两个一样)是个函数. 
 
 		var reqOne = require.config({
  			context: "version1",
  			baseUrl: "version1"
		});

		reqOne(["require", "alpha", "beta",],function(require,   alpha,   beta) {
 			 	log("alpha version is: " + alpha.version); //prints 1
  				log("beta version is: " + beta.version); //prints 1

  			setTimeout(function() {
  				//延迟加载,需要在前面依赖'require'
   				require(["omega"],function(omega) {
        			log("version1 omega loaded with version: " +omega.version); 
        			//prints 1
      				}
    			);
  			}, 100);
		});

		var reqTwo = require.config({
      		context: "version2",
      		baseUrl: "version2"
    	});

		reqTwo(["require", "alpha", "beta"],function(require,   alpha,   beta) {
  				log("alpha version is: " + alpha.version); //prints 2
  				log("beta version is: " + beta.version); //prints 2

  			setTimeout(function() {
  				//延迟加载,需要在前面依赖'require'
    			require(["omega"],function(omega) {
        			log("version2 omega loaded with version: " +omega.version); 
        			//prints 2
      			}
    			);
  			}, 100);
		});
 
 如果没有指定“require”依赖，则很可能会出现错误。
 
 paths备错配置
 ---
 
 paths配置如下:
 
 		paths: {
        	jquery: [
            	'http://ajax.googleapis.com/ajax/libs/jquery/1.4.4/jquery.min',
            	/如果没找到CDN则加载下面位置的jquery
            	'lib/jquery'
        	]
    	}
 
 
 
 页面加载控制及DOM Ready事件监听
 ---
 需要与DOM交互的工作应等待DOM Ready。现代的浏览器通过DOMContentLoaded事件来知会。但是，不是所有的浏览器都支持DOMContentLoaded。**domReady模块**实现了一个跨浏览器的方法来判定何时DOM已经ready。下载并在你的项目中如此用它：
 
 		require(['domReady'], function (domReady) {
  			domReady(function () {
    			//Dom ready后调才执行这里的函数
    			//需要操作DOM的代码放置在这里
    		
  			});
		});
	
		//ps,可以用下面的结构,与主结构分开,避免耗时导致其他js不执行
	
		var reqDOM = requirejs.config({....});
		reqDOM(['domReady'],function(domReady){
			domReady(function () {
    			//Dom ready后调才执行这里的函数
    			//需要操作DOM的代码放置在这里
    		
  			});
		});

requirejs, require, define的区别
---
>requirejs, require两个使用起来完全一样, 也找不到区别的方法....  
requirejs.config()或者 require.config()   

define和require在依赖处理和回调执行上都是一样的，不一样的地方是**define的回调函数需要有return语句返回模块对象**，这样define定义的模块才能被其他模块引用；**require的回调函数不需要return语句**。
>著作权归作者所有。  
商业转载请联系作者获得授权，非商业转载请注明出处。  
作者：Xin You  
链接：http://www.zhihu.com/question/21260764/answer/50194362  
来源：知乎  
	
所以,**define定义require加载使用，这个理解是对的。**



	define([require,path/myMod1,path/moMod2], function(require){  
    	var mod1 = require('path/myMod1'), mod2 = require('path/myMod2');
	})

	其实逻辑上类似于

	define([require], function(require){  
    	var mod1 = require('path/myMod1'), mod2 = require('path/myMod2');
	})

	你只是把参数丢了而已，所以这样也行

	define([require,path/myMod1,path/moMod2], function(require){  
    	var mod1 = arguments[1], mod2 = arguments[2];
	})

	所以，可以这么用

	define([require,path/myMod1,path/moMod2], function(require,mod1,mod2){  
    	// 这里 mod1 和 mod2 都准备好了，还可以用require继续加载别的
	})

	或者，改成你看到的那些例子

	define([path/myMod1,path/moMod2], function(mod1,mod2){  
	})



	所以

	define(['jquery.validate'], function(validate){
    	// 这里可以用validate，但是用不了jquery了？
	})

	这样本质上并没有问题，只不过你引用了这个插件，必然也是想用jquery的而不只是validate，

	所以

	define(['jquery', 'jquery.validate'], function(jquery, validate){
    	// 这样才有jquery
	})

	像你一开始那样使用也行
	
	define(['require', 'jquery.validate'], function(require, validate){
    	var jquery = require('jquery');
	})


