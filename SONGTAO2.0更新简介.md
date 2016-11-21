# SONGTAO2.0更新简介

> 很抱歉，博客2.0更新已经快一个星期了，但是文档说明现在才总结完毕。😋😋


本次版本更新主要是对VUE版本更新，由之前的1.0替换为现在的2.0。另外还包括与其共同工作的插件。本文分为两部分，一是对VUE2.0升级过程中遇到的问题进行总结，另一个是对版本功能更新的介绍。博客前后台代码已同步到GitHub，欢迎Star学习！

下面对部分语法改变的部分进行说明，涉及的面不太广，改写难度不大。下面是遇到的问题和改写注意点，仅供参考。


## VUE2.0升级过程中遇到的问题

### vue2.0语法部分

列表数据渲染

```
1.0版本：
v-for="article of articleList" track-by="$index"
2.0版本：
v-for="article of articleList" :key="article._id"
```

filter语法

```
1.0版本：
<span>{{article.publish_time   | moment "from" "now"}}</span>
2.0版本：需要加到括号里面
<span>{{article.publish_time  | moment("from","now")}}</span>
```

属性

```
1.0版本：
<a href="'mailto:{{item.email}}">{{item.email}}</a>
2.0版本：
<a :href="'mailto:'+item.email">{{item.email}}</a>
```

事件

```
2.0版本移除，此属性，事件直接在DOM绑定
events: {
  'replyThisComment': function (data) {}
 }
```

父组件子组件通信机理：

```
父组件通过prop传递数据，子组件通过事件通知父组件，子组件的状态使用vuex管理
<comment-box  
          :article-id="comment.article_id" 
          :pre-id="comment._id" 
          @event="eventhandler">
</comment-box>
```





### vue-router部分

> router这部分的代码修改很多，不懂的参考我写的[代码](https://github.com/xiangsongtao/X-SONGTAO-VUE/blob/master/src/router.js)，这部分让我想到了分形。。



$router部分方法修改

```
版本1.0：go只能传递数值
_this.$router.go({
      name: 'index'
});
版本2.0
_this.$router.push({
      name: 'index'
});
```

路由跳转部分书写

```
版本1.0：
<a v-link="{ name: 'artList',query: { listType: 'latest' }, activeClass:'active'}">
    最新列表
</a>
版本2.0
<router-link :to="{ name: 'artList',query: { listType: 'latest' }}" activeClass="active" tag="a">
    最新列表
</router-link>
```

子路由的path不带```\```，

```
{
    path: 'tag-list',
    name: 'tagList',
    redirect: '/tag-list/classify',
    component: {
        template: '<router-view></router-view>'
    },
    children: [
        {
            path: 'classify',
            //自路径没有反斜杠name: 'tagListClassify',
            component: require('./views/blog.tagList.vue')
        },
        {
            path: 'find-by-tag-id',
            name: 'tagListFindByTagId',
            component: function(resolve){
                require([
                    './views/blog.articleList.vue'
                ],
                resolve)
            }
        }
    ]
}
```

$router全局化，方便外部js操作路由。例如，权限操作或者错误处理跳转等。

```
/**
 * $router全局化，便于外部js调用，
 * 使用$router操作路由不会出现一闪的情况
 * */
window.$router = router;

//other.js
window.$router.replace({
      name:'login'
});
 window.$router.back();
```

### vuex部分

```
1.0移除部分
vuex: {
	getters: {
		isPlaying: state=>state.isPlaying,
			isLoading: state=>state.isLoading,
			currentMusicInfo: state=>state.currentMusicInfo,
			duration: state=>state.duration,
			rightNow: state=>state.rightNow,
			rightPercent: state=>state.rightPercent,
			canAutoPlay: state=>state.canAutoPlay,
	},
	actions: {
		setCanAutoPlay,
	}
}

2.0语法：
import {mapState,mapActions} from 'vuex';

//在computed中列出需要的状态
computed:{
...mapState({
		isPlaying: 'isPlaying',
		isLoading: 'isLoading',
		currentMusicInfo: 'currentMusicInfo',
		duration: 'duration',
		rightNow: 'rightNow',
		rightPercent: 'rightPercent',
		canAutoPlay: 'canAutoPlay',
	}),
}

//在methods中列出需要的方法
methods: {
...mapActions({
		setCanAutoPlay: 'setCanAutoPlay'
	}),
}
```


在外部js中调用vuex中的方法实现状态读取及修改。
例如权限修改及错误通知

```
// 如何在vue外面使用action和store
import store from './vuex/store'
store.state.isLogin//状态读取
store.dispatch('setLoginState',true)，//action中的方法
```


### 最佳实践

1. created钩子中获取页面渲染的数据
2. mounted中不定义function，function定义要写在method中
3. 本地this使用_this缓存
4. 数据处理好后再显示在DOM中，而不是在DOM中使用filter
5. 如果涉及到列表展示的话，需要在computed中定义一个orderedData或者displayData用于数据筛选




### 其余部分
- vStorage未修改，这部分插件的文档还在整理，地址[在这里](https://github.com/xiangsongtao/vStorage)
- vue-multiselect插件升级，最终已测试使用的版本地址[在这里](https://github.com/xiangsongtao/X-SONGTAO-VUE/plugin/vue-multiselect)。
- Resource未修改



## 新功能介绍及改进

### 打开“我的简介”动画处理

博客是全响应式的设计，故动画会因为适配窗口的大小而完全不同。使用mediaQuery改变不同窗口的样式达到此效果，另外，vue的transition的使用是有条件的，因此我这里手动实现了动画功能。

### 后台缓存处理

第一个缓存处理，主要是在后台的article文档中增加了abstract和html两个字段，文章在保存时，后台对文章的markdown进行编译得到文章摘要及编译后的html内容，预先保存而不是在访问文章的时候再编译，缩短了文章的访问时间。

第二个缓存处理是针对访问统计的。因为每次打开都进行统计延长了页面数据装载的时间，故在nodejs中自己设计了个键值对缓存系统，可设定缓存过期时间。


### 其余改进
- “我的简介”动画优化
- 手机端样式优化
- 文章编辑自动保存时间改为5s
- 文章评论数重新核对
- 手机模式，不更换壁纸
- 手机模式可评论文章
- 文章详情页文章当页切换

(完)