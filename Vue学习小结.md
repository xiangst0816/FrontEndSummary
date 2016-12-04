# Vue2.0小结

> 相同的内容经过实战的熏陶再次看看会有不同的体验，这里对Vue官方文档再次学习，总结下使用过的与未使用的部分，以期在未来能写出更好的Vue代码。



## Vue构造器内部的事儿

为了方便函数调用及数据调用，在书写Vue的时候都是在new Vue({})进行，我这里主要在函数内这个角度进行总结，希望能开拓你的角度。先上一个大概框架，这也是我在进行项目的时候多次复制粘贴的部分：

```
var vm = new Vue({
//1. 挂载位置

//2. 数据

//3. 模板

//4. 方法

//5. 生命周期钩子

// 计算属性
computed

});
```

对上面的部分我注意讲解：

### 1. 挂载位置

显式的在HTML中定义Vue的作用范围，使用id标明，当然也可以使用class，不过就要写成```el: '.app'```，建议写成id，因为使用id在DOM中查找会更快。可以在HTML中定义多个Vue事例，但是别嵌套定义。

### 2. 数据

在HTML中使用双大括号```{{name}}```绑定data中的数据即可。可能在这里你会有这样的需求：Ajax返回的数据需要forEach处理后显示在页面上，但是数据修改的结果并没实时更新DOM，这里有两个解决办法：



1. Ajax返回的数据先进行forEach处理后在赋值给```this.data```，检查下代码。
2. 使用computed处理，return需要返回```this.data```处理后的结果。


### 3. 模板

按照我的习惯，我都是使用HTML模板而不是类似于JSX的render函数完成数据填充。我认为，好的工具或者框架应该兼顾开发者体验和程序运行效率。与此同时，对框架面对的业务范围应该都能抽象到位，API的设计或者协同组件都能完美互补。他的反例我想就是JSX语法，html和css柔和到js中我认为不是很好的开发体验。

### 4. 方法及事件处理

methods中定义的方法可在vm中也可以在HTML中使用，即“事件”。按照我的编写习惯，我一般都将方法在methods中定义，而不是created或者mounted中。



### 5. 计算属性

计算属性主要是为了简化HTML中绑定数据的逻辑复杂度与阅读难易度而设定的。正如上面所讲，对于需要处理的后再显示的数据使用此方法：

1. 各种格式化(金额/日期)
2. 从data中提取嵌套较深的数据
3. 根据data中某字段判断返回true/false等。


> 以上，请注意```return```的写法，其直接返回修改数据的this.data的修改值。

在{{}}中使用函数返回修改的值也是可以的。区别在于：计算属性能缓存结果，而函数则每次都进行计算。

故，下面的代码只会执行一次

```
computed: {
  now: function () {
    return Date.now()
  }
}
```

可以对计算属性赋值：

```
computed: {
  fullName: {
    // 取值时 getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // 赋值时，会修改firstName和lastName值 setter 
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
```





### 6. 生命周期钩子

官方的图我认为已经说的很详细了，图片：


以上，使用的比较多的是created和mounted。created用于获取初始化数据的地方；mounted用户操作DOM，比如轮播swiper插件初始化的位置。




### 7. 组件
















## HTML函数模板中发生的事儿

### 1. 数据绑定

> 注意：过滤器 只能在{{}}中使用

```
// 双向绑定
<span>Message: {{ msg }}</span>
// 单项绑定 v-once
<span v-once>This will never change: {{ msg }}</span>
// 绑定HTML
<div v-html="rawHtml"></div>
<div>{{{rawHtml}}}</div>

// 绑定属性
<div v-bind:id="dynamicId"></div>
<div :data-id="index"></div>用于在HTML中绑定自定义属性
<a v-bind:href="url"></a>
<button v-bind:disabled="someDynamicCondition">Button</button>

// 在{{}}中使用js表达式，但是建议使用computed计算完后在回填HTML中，
// 保持逻辑清晰，方便别人阅读代码。
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}

// 过滤器 只能在此使用
{{ message | filterA | filterB }}串联
{{ message | filterA('arg1', arg2) }}带参数，分别是过滤器中的第2、3号参数

// 指令中使用
<p v-if="seen">Now you see me</p>
```

如果第一次使用的话，在数据回填之前，会在HTML中出现双大括号{{}}的文本，有失美观，使用下面的代码在全局css中定义，可解决问题：

```
// 数据渲染完毕后显示内容
[v-cloak] {
    display: none;
}
```
### 2. 方法及事件绑定

``` v-on: click ```或者简写```@ click ```

```
<button v-on:click="counter += 1">增加 1</button>
<!--简写-->
<button @click="counter += 1">增加 1</button>
<!--methods中定义的函数-->
<button @click="counter()">增加 1</button>
<!--传入事件回调 $event-->
<button @click="counter($event)">增加 1</button>
```

####  3. 事件修饰符和按键修饰符
 

 
> 修饰符的存在意义：将函数与DOM事件解耦，即函数只处理数据逻辑，事件修饰符解决与DOM相关操作。当ViewModel 被销毁时，所有的事件处理器都会自动被删除。

```
#事件修饰符
 
.stop       event.stopPropagation()
.prevent    event.preventDefault()
.capture    添加事件侦听器时使用事件捕获模式
.self       只当事件在该元素本身（而不是子元素）触发时触发回调

<!-- 示例 -->
<a v-on:click.stop="doThis"></a>

#按键修饰符

.enter
.tab
.delete (捕获 “删除” 和 “退格” 键)
.esc
.space
.up
.down
.left
.right

<!-- 示例 -->
<input v-on:keyup.13="submit">
<input v-on:keyup.enter="submit">
```


### 4. Class和Style中绑定

总体来说，就是由data中的属性值修改HTML中的class或者style，共有3中方式。

```
<!--1. 样式class 传入对象 classObject = { active: isActive }-->
<div v-bind:class="classObject"></div>

<!--2. 样式class-->
<div v-bind:class="{ active: isActive }"></div>
<div class="static" v-bind:class="{ active: isActive, 'text-danger': hasError }"></div>

<!--3. 样式class 使用array语法-->
<!--activeClass，errorClass是data中的属性值，动态设置class名称-->
<div v-bind:class="[activeClass, errorClass]">
<div v-bind:class="[isActive ? activeClass : '', errorClass]">
<div v-bind:class="[{ active: isActive }, errorClass]">
```

```
<!--1. 样式style 传入对象 -->
<!--styleObject = { color: activeColor, fontSize: fontSize + 'px' }-->
<div v-bind:class="styleObject"></div>

<!--2. 样式style-->
<!--activeColor，fontSize是data中的属性值，动态设置class名称-->
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>

<!--3. 样式style 使用array语法-->
<!--baseStyles，overridingStyles是data中的定义的样式对象-->
<div v-bind:style="[baseStyles, overridingStyles]">
<!--自动加前缀，例如transform-->
```

### 5. 条件渲染

和angular很相似，v-if, v-else/v-show，v-show会在页面初始化的时候渲染内部内容，而v-if不会。故v-if适用在外部数据渲染前隐藏模板内容，这样不会因为data未准备好而报错。


### 6. 列表渲染

可以渲染三种类型的数据：array、object、number。

```
<!--数组列表-->
<li v-for="item in items"></li>
<li v-for="(item,index)in items"></li>

<!--对象-->
<li v-for="value in object"></li>
<li v-for="(value, key, index) in object"></li>

<!--数字-->
<span v-for="n in 10">{{ n }}</span>

<!--绑定顺序 key为通用机制-->
<div v-for="item in items" :key="item.id">
  <!-- 内容 -->
</div>
```

数组有些特殊，因为有些数组的方法是在原值上修改，有些是返回新的数组，这里需要注意：如果是返回新的数组，请在将其复制给原数组，保持响应特性。

Array原值修改的方法：

```
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```

返回新Array的方法：

```
filter()
concat()
slice()
```

以下设置需要注意：

|  使用场景  |  避免  |  推荐  |
|:--------:|:-----:|:-----:|
| 直接设置一项的索引时 | ```vm.items[indexOfItem] = newValue``` | ```Vue.set(example1.items, indexOfItem, newValue)``` ```example1.items.splice(indexOfItem, 1, newValue)```|
| 修改数组的长度时 | ```vm.items.length = newLength``` | ```example1.items.splice(newLength)```|


### 7. 表单控件绑定

选择列表中的value不只能绑定String结果，也能能绑定对象。选择列表+列表循环。

```
<select v-model="selected">
    <!-- 内联对象字面量 -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>

<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{ option.text }}
  </option>
</select>


```

修饰符

```
.lazy        在 change 事件中双向同步数据（区别debounce）使用在,比如输入的值会进行复杂计算或者异步请求
.number        返回number值
.trim        去除首尾空格

<input v-model.number ="age" type="number">
<input v-model.trim="msg">
```


## 关于组件

这一节需要单独说明，



