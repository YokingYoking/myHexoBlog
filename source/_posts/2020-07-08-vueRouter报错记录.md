---
title:      vue-router.push()的控制台报错问题
date:       2020-07-08
categories:
    - 知识整理
tags:
    - 前端
    - JavaScript
    - Vue
    - vue-router
    - 报错解决
excerpt: NavigationDuplicated报错
---
# vue-router.push()的控制台报错问题
## 报错信息
当重复点击路由导航或者在进入路由后再进行router.push拼接参数时，会出现NavigationDuplicated报错，字面意思就是导航重复了。
## 报错原因
实际上这个报错对功能不会有影响，但是控制台报错确实不太好看，通过查阅官方文档之后会看到router的方法里除了传入的参数（path/query等），还有两个回调函数onComplete，onAbort，分别是在成功和出现问题的时候回调，而会报这个错是vue-router内部的一些问题，因为一般调用的时候不会填写这两个回调函数的参数所以报错直接被全局截获报出错误。查阅了其他人同样遇到这个错误的情况后得知是由于vue-router3.1版本把push/replace等方法改写成了promise，onComplete和onAbort两个参数的存在也解释的通了。而且这个问题在[vue-router的github issue](https://github.com/vuejs/vue-router/issues/2872)上也有提到，可以参考一下开发者的回答。
从webstorm的提示中也能看得见这是个promise。
![rBAoMF8GjuSAO4gCAAA4btJx-nA006.png](https://i.loli.net/2020/07/15/FD8VQIOTw2PqM5p.png)
## 解决方法
正如开发者在github上的回答，我们只需要捕获错误进行自己的处理，不让它暴露到全局去就不会再控制台报错，解决方法如下：

```javascript
router.push('/location', () => {})// orrouter.push('/location').catch(err => {})
```

但我们在实际应用时不可能每个router.push都加一个参数，过于麻烦，所以我们直接在改写应用的router底层方法：

```javascript
// 在router/index.js文件中import VueRouter from 'vue-router' 
// 一般我们都在这个文件中引入，总之在哪里引入就在哪里加这段代码去改写
const originalPush = VueRouter.prototype.pushVueRouter.prototype.push = function push(location) {  
    return originalPush.call(this, location).catch(err => err)
    }
    const routes = [] // 你的路由表
    const reouter = new Router({routes})
    export default router
```

但实际上这样改写当router真的出现其他错误的时候同样也不会被抛出到控制台，是有缺陷的，因此这当然不是一个好的解决方法，后续只能等vue-router开发的大神在4.x的版本中进行改进了。
## 参考链接
[csdnblog1](https://blog.csdn.net/gxdvip/article/details/101016946)[csdnblog2](https://blog.csdn.net/XuM222222/article/details/104002534)
