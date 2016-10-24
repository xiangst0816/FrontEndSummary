# JS深浅拷贝相关

在写代码的时候翻看源码无意中看到了对象的深浅复制的代码，估计以后也会用的上，虽然徒手从零开始写有些困难，但是使用的时候能快速拿出来也是好的。下面是对深浅复制的总结笔记！

## 先说下基础类型和引用类型

下面这张图是对基础类型和引用类型的分类

![](http://xiangsongtao.com/uploads/1474693606000.png "")

在 JS 中有一些基本类型像是```Number```、```String```、```Boolean```，而对象就是像这样的东西```{ name: 'Larry', skill: 'Node.js' }```，对象跟基本类型最大的不同就在于他们的传值方式。

基本类型是传 value，像是这样：

```
var a = 10;
var b = a;
b = 20;
console.log(a);//10
console.log(b);//20
```

在修改```a```时并不会改到```b```

但对象就不同，对象传的是reference：

```
var obj1 = { a: 10, b: 20, c: 30 };
var obj2 = obj1;
obj2.b = 100;
console.log(obj1);
// { a: 10, b: 100, c: 30 } <-- b 被改到了
console.log(obj2);
// { a: 10, b: 100, c: 30 }
```
复制一份```obj1```叫做```obj2```，然后把```obj2.b```改成100，但却不小心改到```obj1.b```，因为他们根本是同一个对象，这就是所谓的**浅拷贝**。

要避免这样的错误发生就要写成这样：

```
var obj1 = { a: 10, b: 20, c: 30 };
var obj2 = { a: obj1.a, b: obj1.b, c: obj1.c };
obj2.b = 100;
console.log(obj1);
// { a: 10, b: 20, c: 30 } <-- b 沒被改到
console.log(obj2);
// { a: 10, b: 100, c: 30 }
```

## 浅拷贝(Shallow Copy) VS 深拷贝(Deep Copy)

浅拷贝只复制指向某个对象的**指针**，而不复制对象本身，新旧对象还是共享同一块内存。但深拷贝会另外创造**一个一模一样的对象**，新对象跟原对象不共享内存，修改新对象不会改到原对象。

## Object.assign ES6 的新函数

用法如下：
ta
```
var obj1 = { a: 10, b: 20, c: 30 };
var obj2 = Object.assign({}, obj1);
obj2.b = 100;
console.log(obj1);
// { a: 10, b: 20, c: 30 } <-- 沒被改到
console.log(obj2);
// { a: 10, b: 100, c: 30 }
```

但是，只能处理深度只有一层的对象，没办法做到真正的 Deep Copy。不过如果要复制的对象只有一层的话可以考虑使用它。

## 处理单纯的数据对象时(不能处理有函数的情况)

用```JSON.stringify```把对象转成字符串，再用```JSON.parse```把字符串转成新的对象。


```
var obj1 = { body: { a: 10 } };
var obj2 = JSON.parse(JSON.stringify(obj1));
obj2.body.a = 20;
console.log(obj1);
// { body: { a: 10 } } <-- 沒被改到
console.log(obj2);
// { body: { a: 20 } }
console.log(obj1 === obj2);
// false
console.log(obj1.body === obj2.body);
// false
```

## 库函数实现 Deep Copy

### jquery

jquery中提供的一个函数```$.extend```可以用来做 Deep Copy。

```
var $ = require('jquery');
var obj1 = {
    a: 1,
    b: { f: { g: 1 } },
    c: [1, 2, 3]
};
var obj2 = $.extend(true, {}, obj1);
console.log(obj1.b.f === obj2.b.f);
// false
```
### lodash

lodash提供```_.cloneDeep```用来做 Deep Copy。

```
var _ = require('lodash');
var obj1 = {
    a: 1,
    b: { f: { g: 1 } },
    c: [1, 2, 3]
};
var obj2 = _.cloneDeep(obj1);
console.log(obj1.b.f === obj2.b.f);
// false
```

## 哥哥给你手写一个

```
var obj1 = {
    a: 1,
    b: { f: { g: 1 } },
    c: [1, 2, 3]
};
var obj2 = {} ;
_merge(true,obj2,obj1);
console.log(obj2)

function _merge(deep,target, source) {
    for (var key in source) {
        if (deep && (isPlainObject(source[key]) || isArray(source[key]))) {
            if (isPlainObject(source[key]) && !isPlainObject(target[key])) {
                target[key] = {};
            }
            if (isArray(source[key]) && !isArray(target[key])) {
                target[key] = [];
            }
            _merge(deep,target[key], source[key]);
        } else if (source[key] !== undefined) {
            target[key] = source[key];
        }
    }
}

function isPlainObject(obj) {
    return isObject(obj) && Object.getPrototypeOf(obj) == Object.prototype;
}
function isObject(obj) {
    return obj !== null && typeof obj === 'object';
}
function isArray(arr) {
    return Array.isArray(arr);
}
```



> 以上就是全部了，也不算难，多注意多留心就好。

