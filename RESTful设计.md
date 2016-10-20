# RESTful API

## RESTful架构

REST是Representational State Transfer的缩写，表示“表现层状态转移”。这个其实是在讲一个过程。

很久之前，用户访问服务器的时候，服务器是主角。他根据用户的请求决定发送设么东西给客户端，而客户端只有接收的“命”；

现在REST来了，主角现在是用户，用户通过HTTP协议中的多种请求状态（GET、POST、PUT、DELETE等），要求服务器按照URL中的参数将资源返回。这时服务器就像一个静态的资源服务器一样，每一个URI代表一种资源。

```
RESRful：
资源 = URI + Method

Method：
GET用来获取资源
POST用来新建资源（也可以用于更新资源）
PUT用来更新资源
DELETE用来删除资源

```

因此，操作资源的动作只能由GET、POST、PUT、DELETE等进行，URI代表资源的名称，故设计URI时尽量使用名词！








## 参考

- [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)