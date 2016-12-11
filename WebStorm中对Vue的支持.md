# WebStorm中对vue的支持

这篇文章主要介绍了如何在WebStorm中进行配置达到对Vue的完美支持，包括代码检查、自动排版等。

## Plugins安装

首先两个插件是必不可缺少的，这个需要在WebStorm->Preferences->Plugins中搜索'vue-for-idea'插件。另外'Vue.js'这个插件不好用，别安装！！


## 默认模板修改

依次进入顺序：

> WebStorm->Preferences->Editor->Code Stype->File andCode Templates


之后选择Vue File，下面是我的模板，制定style和script的类型是为了能够自动格式化。

```html
<template>
    <div class="main">
        <header-component/>
        <div>this is template body</div>
        <other-component/>
    </div>
</template>
<style scoped lang="scss">
    body{
        background-color:#ff0000;
    }
</style>
<script type="text/ecmascript-6">
    import HeaderComponent from './components/header.vue'
    import OtherComponent from './components/other.vue'
    export default{
        data(){
            return{
                msg:'hello vue'
            }
        },
        components:{
            'other-component':OtherComponent,
            HeaderComponent,
        }
    }
</script>
```

## 注册文件模板

依次进入顺序：

>  WebStorm->Preferences->Editor->Code Stype->File Types

之后选择Vue，在下面添加对vue的支持，输入：```*.vue```。如果已经默认改好了，那就不修改了。

## Script代码片段绿色背景

完成以上的设置后，会在Script代码片段处出现绿色的背景提示，虽然不影响排版和代码错误检查，但是影响编程心情，问题如下：

![](http://xiangsongtao.com/uploads/1481340380000.png)



这个问题也一直困扰了我很久，在网上也没找到答案，偶然一次在查看代码片段配色的时候找到了这个设置，这里分享下，依次点击进入：

> WebStorm->Preferences->Editor->Color&Fonts->General->Code->inject language fragment->Background

Background不设置颜色，之后Apply就好了。

![](http://xiangsongtao.com/uploads/1481340545000.png)


（完）
