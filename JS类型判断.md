# JS类型判断

## 常用的类型判断 typeof

在进行类型判断时，第一反应是使用typeof来做，写法：

```
function type(value){
    return typeof value
}

console.log(type ('str'))
output  'string'  
```

但是这个方法可返回的类型只有以下几种（注意都是小写）：

```object```(对象,数组,日期和null)、 ```string```、```number```、```function```、```undefined```、```boolean```

> 需要特别注意的是，使用typeof判断未定义的变量不会抛出异常```Uncaught ReferenceError: xxx is not defined```，但是其余方法都会！


## instanceof 判断

instanceof 用于判断一个变量是否某个对象的实例，是一个三目运算式

这里的instanceof测试的object是指js语法中的object，不是指dom模型对象

```
//之后就是对array、object、Date进行区分了
({}) instanceof Object
-> true

([]) instanceof Array
-> true

(new Date()) instanceof Date
-> true

(function(){}) instanceof Function
-> true

//Null String Number Boolean 判断都是不行的。
```

## constructor 判断

constructor 属性返回对象相对应的构造函数，基础类型会临时包裹为对象！



```
({}).constructor === Object
-> true

([]).constructor === Array
-> true

(new Date()).constructor === Date
-> true

('str').constructor === String
-> true

(123).constructor === Number
-> true

(true).constructor === Boolean
-> true

(function(){}).constructor === Function
-> true

//undefined、 Null 判断都是不行的。
```
## 类型判断全家福

这部分摘自```vue-resource```源码

```
function isDefined(value) {
    return value !== undefined && value !== null;
}
var isArray = Array.isArray;

function isString(val) {
    return typeof val === 'string';
}

function isBoolean(val) {
    return val === true || val === false;
}

function isFunction(val) {
    return typeof val === 'function';
}

// data、object、array
function isObject(obj) {
    return obj !== null && typeof obj === 'object';
}

// 纯对象 object
function isPlainObject(obj) {
    return isObject(obj) && Object.getPrototypeOf(obj) == Object.prototype;
}

function isFormData(obj) {
    return typeof FormData !== 'undefined' && obj instanceof FormData;
}
```



## 终极写法

自己实现了getType函数支持各种类型的判断，包括：```string```、 ```date```、 ```number```、 ```array```、 ```object```、 ```boolean```、 ```null```、 ```undefined```，判断后返回以上字符串。


```
function getType(value) {
    return Object.prototype.toString.call(value).match(/^(\[object )(\w+)\]$/i)[2].toLowerCase()
}

getType('str') === 'string';
getType(new Date()) === 'date';
getType(123) === 'number';
getType([]) === 'array';
getType({}) === 'object';
getType(true) === 'boolean';
getType(!!"str") === 'boolean';
getType(null) === 'null';
getType(undefined) === 'undefined';
var str;
getType(str) === 'undefined';

//Output true
```

（完）

