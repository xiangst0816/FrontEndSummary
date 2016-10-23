# JavaScript arguments 对象全面介绍

## 什么是 arguments

arguments 是一个类数组的对象，代表传给一个function的参数列表。

```
function printArgs() {
    console.log(arguments);
}

printArgs("A", "a", 0, { foo: "Hello, arguments" })；
执行结果是：
["A", "a", 0, Object]
```
## arguments 操作

### arguments length

arguments 是个类数组对象，其包含一个 length 属性，可以用 arguments.length 来获得传入函数的参数个数。

```
function func() {
    console.log("The number of parameters is " + arguments.length);
}
```


### arguments 转数组

```
Array.prototype.slice.call(arguments);
[].slice.call(arguments);
```



### arguments 传递出去
将函数的 arguments 对象泄露出去了，最终的结果就是 V8 引擎将会跳过优化，导致相当大的性能损失。

```
// Leaking arguments example1:
function getArgs() {
    return arguments;
}
// good arguments example1:
function getArgs() {
    const args = [].slice.call(arguments);//slice返回新数组
    return args;
}

```
### 修改 arguments 值

在严格模式与非严格模式下，修改函数参数值表现的结果不一样。在严格模式下，函数中的参数与 arguments 对象没有联系，修改一个值不会改变另一个值。而在非严格模式下，两个会同步影响。

### 将参数从一个函数传递到另一个函数

下面是将参数从一个函数传递到另一个函数的推荐做法。

```
function foo() {
    bar.apply(this, arguments);
}
function bar(a, b, c) {
    // logic
}
```
### arguments 与重载

利用 arguments 模拟重载。

```
function add(num1, num2, num3) {
    if (arguments.length === 2) {
        console.log("Result is " + (num1 + num2));
    }
    else if (arguments.length === 3) {
        console.log("Result is " + (num1 + num2 + num3));
    }
}
```

## ES6 中的 arguments

### 扩展操作符

```
function func() {
    console.log(...arguments);
}

func(1, 2, 3);
执行结果是：
1 2 3
```

### Rest 参数

```
function func(firstArg, ...restArgs) {
    console.log(Array.isArray(restArgs));
    console.log(firstArg, restArgs);
}

func(1, 2, 3);
执行结果是：
true
1 [2, 3]
```
从上面的结果可以看出，Rest 参数表示除了明确指定剩下的参数集合，类型是 Array。

### 默认参数

```
function func(firstArg = 0, secondArg = 1) {
    console.log(arguments[0], arguments[1]);
    console.log(firstArg, secondArg);
}

func(99);
执行结果是：
99 undefined
99 1
```
可见，默认参数对 arguments 没有影响，arguments 还是仅仅表示调用函数时所传入的所有参数。


### arguments 转数组

Array.from() 是个非常推荐的方法，其可以将所有类数组对象转换成数组。



## 参考

- [JavaScript arguments 对象全面介绍](https://segmentfault.com/a/1190000007091243)







