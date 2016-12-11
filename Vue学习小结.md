# Vue2.0小结

> 相同的内容经过实战的熏陶再次看看会有不同的体验，这里对Vue官方文档再次学习，总结下使用过的与未使用的部分，以期在未来能写出更好的Vue代码。

## Vue构造器内部的事儿

为了方便函数调用及数据调用，在书写Vue的时候都是在new Vue({})进行，我这里主要在函数内这个角度进行总结，希望能开拓你的角度。先上一个大概框架，这也是我在进行项目的时候多次复制粘贴的部分，之后我对下面的部分我逐一讲解：

```javascript
var vm = new Vue({
    //1. 挂载位置
    el: '#app',
    //2. 数据
    data: {
        name: 'songtao',
        data: [], //数据
    },
    filters: {
        // 第1个参数是传入过滤的值，第2、3是参数
        capitalize: function (value, arg1, arg2) {
            if (!value) return ''
            value = value.toString()
            return value.charAt(0).toUpperCase() + value.slice(1)
        }
    },
    //3. 数据watch
    watch: {
        // 如果 question 发生改变，这个函数就会运行
        question: function (newQuestion) {
            this.answer = 'Waiting for you to stop typing...'
            this.getAnswer()
        }
    },
    //4. 计算属性-
    computed: {
        // a computed getter
        reversedMessage: function () {
            // `this` points to the vm instance
            return this.message.split('').reverse().join('')
        },
        fullName: {
            // getter
            get: function () {
                return this.firstName + ' ' + this.lastName
            },
            // setter
            set: function (newValue) {
                var names = newValue.split(' ')
                this.firstName = names[0]
                this.lastName = names[names.length - 1]
            }
        }
    },
    //5. 方法及事件处理
    methods: {
        say: function (message) {
            alert(message)
        },
        // <button v-on:click="warn('Form cannot be submitted yet.', $event)">Submit</button>
        warn: function (message, event) {
            // 现在我们可以访问原生事件对象
            if (event) event.preventDefault()
            alert(message)
        }
    },

    //6. 生命周期钩子
    beforeCreate: function () {},
    created: function () {
        console.log('获取数据')
    },
    beforeMount: function () {},
    mounted: function () {
         console.log('操作DOM')
    },
    beforeUpdate: function () {},
    updated: function () {},
    beforeDestroy: function () {},
    destroyed: function () {},

    //7. 组件
    components: {
        // <my-component> 将只在父模板可用
        'my-component': {
            template: '<div>A custom component!</div>'
        }
    }
});

```

### 1. 挂载位置

显式的在HTML中定义Vue的作用范围，使用id标明，当然也可以使用class，不过就要写成```el: '.app'```，建议写成id，因为使用id在DOM中查找会更快。可以在HTML中定义多个Vue事例，但是别嵌套定义。

### 2. 数据

在HTML中使用双大括号```{{name}}```绑定data中的数据即可。可能在这里你会有这样的需求：Ajax返回的数据需要forEach处理后显示在页面上，但是数据修改的结果并没实时更新DOM，这里有两个解决办法：

1. Ajax返回的数据先进行forEach处理后在赋值给```this.data```，检查下代码。
2. 使用computed处理，return需要返回```this.data```处理后的结果。


### 3. 模板

按照我的习惯，我都是使用HTML模板而不是类似于JSX的render函数完成数据填充。我认为，好的工具或者框架应该兼顾开发者体验和程序运行效率。与此同时，对框架面对的业务范围应该都能抽象到位，API的设计或者协同组件都能完美互补。他的反例我想就是JSX语法，html和css柔和到js中我认为不是很好的开发体验。

### 4. 方法及事件处理

methods中定义的方法可在vm中也可以在HTML中使用（所谓的“事件”）。按照我的编写习惯，我一般都将方法在methods中定义，而不是created或者mounted中。

### 5. 计算属性

计算属性主要是为了简化HTML中绑定数据的逻辑复杂度与阅读难易度而设定的。正如上面所讲，对于需要处理的后再显示的数据使用此方法：

1. 各种格式化(金额/日期)
2. 从data中提取嵌套较深的数据
3. 根据data中某字段判断返回true/false等。


> 以上，请注意```return```的写法，其直接返回修改数据的this.data的修改值。

在{{}}中使用函数返回修改的值也是可以的。区别在于：计算属性能缓存结果，而函数则每次都进行计算。

故，下面的代码只会执行一次

```javascript
computed: {
  now: function () {
    return Date.now()
  }
}
```

可以对计算属性赋值：

```javascript
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

官方的图我认为已经说的很详细了，这里有图：

<div style="width:100%;text-align:center;">
<img src="http://xiangsongtao.com/uploads/1480993223000.png" width = "400" alt="生命周期钩子" align=center />
</div>

这个是中文说明：

![](http://xiangsongtao.com/uploads/1480993292000.png)

以上，使用的比较多的是created和mounted。created用于获取初始化数据的地方；mounted用户操作DOM，比如轮播swiper插件初始化的位置。




### 7. 组件

见下面单独的说明。



## HTML函数模板中发生的事儿

### 1. 数据绑定

> 注意：过滤器 只能在{{}}中使用

```html
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

```css
// 数据渲染完毕后显示内容
[v-cloak] {
    display: none;
}
```
### 2. 方法及事件绑定

``` v-on: click ```或者简写```@ click ```

```html
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

```javascript
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

```html
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

```html
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

> 在开发中还遇到一个问题，这个问题只在IOS中发现：使用v-show隐藏的页面，其中的methods定义的方法，在使用fastclick处理click事件的时候，点击两次才会触发一次的问题。这个没找到原因，改成了使用```:class```隐藏的手法可以避免。


### 6. 列表渲染

可以渲染三种类型的数据：array、object、number。

```html
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

```javascript
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
```

返回新Array的方法：

```javascript
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

选择列表中的value不只能绑定String结果，也能绑定对象。
> 注意：在预选定的需求中，需要将列表中的某值给selected才能预选中，而不是创造一个值（列表中的item为对象的形式-object） 。

```html
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

```text
.lazy        在 change 事件中双向同步数据（区别debounce）使用在,比如输入的值会进行复杂计算或者异步请求
.number      返回number值
.trim        去除首尾空格

<input v-model.number ="age" type="number">
<input v-model.trim="msg">
```


## 关于组件

这一节需要单独说明，因为这个是文档中说的最强大的部分。

关于父子组件交互的部分这张图已经说明了一切。即，父组件传给子组件数据，子组件给父组件自定义事件响应。保证了每个组件可以在相对隔离的环境中书写和理解，也大幅提高了组件的可维护性和可重用性。

> 类似于这样的一个场景：老子给小子钱买“海天牌”食用盐，买到后放到厨房，小子买到后说一声“全部办完”，此过程老子不管小子是怎么买的。

<div style="width:100%;text-align:center;">
 <img src="http://xiangsongtao.com/uploads/1480992921000.png" width = "400" alt="图片名称" align=center />
</div>

Vue组件的核心思想就三部分：数据(props)、事件(event)、内容分发(slot)。

1. prop：允许外部环境传递数据给组件；
2. event：允许组件触发外部环境的 action；
3. slot：允许外部环境插入内容到组件的视图结构内。

其内容在下面是整体代码中会集中体现，再之后是注意点及解读。

```javascript

Vue.component('child', {
    // data必须是函数，因为对象是引用类型，会在不同组件间共用
    data: function () {
        return {
            counter: 0,
            reversedMyMessage1: this.myMessage.split('').reverse().join(),
        }
    },
    template: '<span>{{ myMessage }}</span>',
    // 简单模式
    props: ['myMessage'],
    // 完成模式
    props: {
        // 基础类型检测 （`null` 意思是任何类型都可以）
        propA: Number,
        // 多种类型 (1.0.21+)
        propM: [String, Number],
        // 必需且是字符串
        propB: {
            type: String,
            required: true
        },
        // 数字，有默认值
        propC: {
            type: Number,
            default: 100
        },
        // 对象/数组的默认值应当由一个函数返回
        propD: {
            type: Object,
            default: function () {
                return {
                    msg: 'hello'
                }
            }
        },
        // 自定义验证函数
        propF: {
            validator: function (value) {
                return value > 10
            }
        },
        // 全家福
        propG: {
            type: Number,
            default: 100,
            required: true,
            validator: function (value) {
                return value > 10
            }
        }
    },
    computed: {
        reversedMyMessage2: function () {
            return this.myMessage.split('').reverse().join()
        },
    },
    methods: {
        increment: function () {
            this.counter += 1
            this.$emit('increment')
        }
    }
})

```

### 1. 使用方法

#### 全局组件

```html
<div id="example">
  <my-component></my-component>
</div>
```

```javascript
// 注册
Vue.component('my-component', {
  template: '<div>A custom component!</div>'
})
// 创建根实例
new Vue({
  el: '#example'
})
```


#### 局部组件

```javascript
var Child = {
  template: '<div>A custom component!</div>'
}
new Vue({
  // ...
  components: {
    // <my-component> 将只在父模板可用
    'my-component': Child
  }
})
```


### 2. 内部数据(data)

 这里的数据必须是function，且返回data数据
 
### 3. 模板(template)

例子已能说明一切了：在组件定义的数据都可以在里面使用，类似于在HTML中绑定数据一样。data和props唯一的区别就是：一个来源内部、一个来源外部。


### 4. 父组件数据(props) 

> 组件实例的作用于是孤立的，这点很明确。每个组件都是一个独立王国。

1. props中定义的外部变量使用驼峰形式(myMessage)，而在HTML中则转化为短横线形式(my-message)
2. 动态的props使用```v-bind:my-message```或者```:my-message```形式
3. props默认传递string类型，即使像number这样的传入。如果想传递number，则使用动态绑定语法```v-bind:some-prop="1"```
4. prop为单向数据流，传入的值建议作为**只读模式**使用，如果要对传入值进行处理，请使用```data```或者```computed```处理
5. 对于应用类型：object、array，请deepCopy后再处理，或者这样：```JSON.parse(JSON.stringify(object))```，只限于数据拷贝


### 5. 事件(event)

1. ```this.$emit('increment')```: 子组件的发出的事件在函数内部定义
2. ```v-on:incrementTotal```: 父组件监听子组件发出的事件
3. 监听原生事件使用native修饰符```<my-component v-on:click.native="doTheThing"></my-component>```
4. 非父子通信可使用eventBus手段，或者vuex

完整父子组件通信过程：


```html
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```

```javascript
Vue.component('button-counter', {
    template: '<button v-on:click="increment">{{ counter }}</button>',
    data: function () {
        return {
            counter: 0
        }
    },
    methods: {
        increment: function () {
            this.counter += 1
            this.$emit('increment')
        }
    },
})
new Vue({
    el: '#counter-event-example',
    data: {
        total: 0
    },
    methods: {
        incrementTotal: function () {
            this.total += 1
        }
    }
})
```

对于v-model这样的自定义表单输入事件怎么写：

1. props->value
2. 有输入时触发input事件
3. template做好接收

```html
<currency-input v-model="price"></currency-input>
```

```javascript
Vue.component('currency-input', {
    template: '\
    <span>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)"\
      >\
    </span>\
  ',
    props: ['value'],
    methods: {
        // Instead of updating the value directly, this
        // method is used to format and place constraints
        // on the input's value
        updateValue: function (value) {
            var formattedValue = value
                // Remove whitespace on either side
                .trim()
                // Shorten to 2 decimal places
                .slice(0, value.indexOf('.') + 3)
                // If the value was not already normalized,
                // manually override it to conform
            if (formattedValue !== value) {
                this.$refs.input.value = formattedValue
            }
            // Emit the number value through the input event
            this.$emit('input', Number(formattedValue))
        }
    }
})
```

### 6. 分发内容(slot)

slot的动作都是在html中下功夫，具体做法：在父组件中，子组件范围内的html内容插入 具有slot标识位置的子组件中。解读有点晦涩，看完代码变化就懂了。

```html
<!--子组件(app-layout)-->
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>


```


```html
<!--父组件-->
<app-layout>
  <h1 slot="header">Here might be a page title</h1>
  <p>A paragraph for the main content.</p>
  <p>And another one.</p>
  <p slot="footer">Here's some contact info</p>
</app-layout>
```

```html
<!--组合结果-->
<div class="container">
  <header>
    <h1>Here might be a page title</h1>
  </header>
  <main>
    <p>A paragraph for the main content.</p>
    <p>And another one.</p>
  </main>
  <footer>
    <p>Here's some contact info</p>
  </footer>
</div>
```


### 7. 其他

1. 使用is特性在组件挂载点动态切换组件
2. 子组件索引```$refs```
3. 组件可异步加载
4. js中组件命名随意，但是在html中只能使用短横线形式


## 高级部分


### 1. 异步队列更新

为了在数据变化之后等待 Vue 完成更新 DOM ，可以在数据变化之后立即使用``` Vue.nextTick(callback)``` 。这样回调在 DOM 更新完成后就会调用。如果是组件内的话，使用``` this.nextTick(callback)```即可。

> 而以前的做法是使用setTimeout给一个很短的定时，是不是有点尴尬。


(完)

