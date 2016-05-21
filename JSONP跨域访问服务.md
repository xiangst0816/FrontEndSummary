JSONP跨域访问服务
===



**跨域访问:**  
>如果我们希望获取的数据和当前页面并不是一个域，著名的同源策略（不同域的客户端脚本在没明确授权的情况下，不能读写对方的资源）会因为安全原因决绝请求，也就是我们不能向其它域直接发送请求以获取资源。  
**iframe、img、style、script等元素**的src属性可以直接向不同域请求资源，jsonp正是利用script标签跨域请求资源的

**异域服务器的php代码:**

		<?php
    		$path=$_SERVER["DOCUMENT_ROOT"].'/books.xml';
    		$json=json_encode(simplexml_load_file($path));
    		$callbackFn=$_GET['callback'];
    		echo "$callbackFn($json);";
		?>	
将xml数据转化成json,之后将传入的callback的值作为函数名传入json数据,之后echo.

**本地callback函数**

		function displayBooks(books){
		****
		}

**本地jsonp请求发送**  
写一个```<script>```标签用于跨域

		function getBooks(){
            var script=document.createElement('script');
            script.setAttribute('type','text/javascript');
            script.setAttribute('src','http://test.com/bookservice.php?			callback=displayBooks');
            document.body.appendChild(script);
        }
        getBooks();
        

**jquery实现**

		function getBooks(){
            $.ajax({
                type:'get',
                url:'http://test.com/bookservice.php',
                dataType:'jsonp',
                jsonp:'callback',
                jsonpCallback:'displayBooks'
            });
        }
    
        
     