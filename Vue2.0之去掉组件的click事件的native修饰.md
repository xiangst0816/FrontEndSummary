# Vue2.0之去掉组件click事件的native修饰

这个是在组件开发中遇到的问题，当时我在编写button的组件，模板是这样的：


```html
<template>
  <button class="disable-hover button ion-button"
          :class="[modeClass,typeClass,shapeClass,sizeClass,colorClass,roleClass,strongClass]">
    <span class="button-inner">
      <slot></slot>
    </span>
    <div class="button-effect"></div>
  </button>
</template>
```

使用是这样子的：

```
<ion-button @click.native="primary()" color="primary">primary</ion-button>
```


根据Vue2.0官方文档关于父子组件通讯的原则，父组件通过prop传递数据给子组件，子组件触发事件给父组件。但父组件想在子组件上监听自己的click的话，需要加上```native```修饰符，故写法就像上面这样。

好吧，处女座的毛病又来了。像button这样常用的组件如果加上native的话是感觉很突兀的。虽然组件设计有自身的考虑，因此我想将click的native去掉，思路如下：

1. 子组件监听父组件给的click事件，
2. 子组件内部处理click事件然后向外发送click事件：```$emit("click".fn)```

改造后的代码如下(亲测可用)：

```html
<template>
  <button class="disable-hover button ion-button" @click="_click"
          :class="[modeClass,typeClass,shapeClass,sizeClass,colorClass,roleClass,strongClass]">
    <span class="button-inner">
      <slot></slot>
    </span>
    <div class="button-effect"></div>
  </button>
</template>

<script type="text/babel">
export default{
    ....
    ....
    methods: {
      _click: function () {
        this.$emit('click', function () {
          alert('inner')
        })
      }
    }
}
</script>
```

父组件中这样使用：

```
<ion-button @click="primary()" color="primary">primary</ion-button>
```

也许读者能看出来，我正在参照IONIC2.X的组件API写Vue2.0的功能组件，目前只完成了：ActionSheet、Button、Icon、Alert、Toast这几个，一边看IONIC源码，一边将思路总结写成Vue代码，也就是花点时间。积累自己的组件库希望以后能开发快点。

现在项目地址在[这里](https://github.com/xiangsongtao/VueMobile)，前期以实现功能为主，不建议用在生产环境，读读代码实现思路就好，中文备注都做好了。

(完)