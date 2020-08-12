---
title:      用async封装axios及其异常处理
date:       2020-06-17
categories:
    - 知识整理
tags:
    - 前端
    - async
    - ES8
    - axios
    - promise
excerpt: 要逐渐用async代替promise了
---
# 用async封装axios

最近项目中原本的请求方法之前写的时候用的还是es6的axios Promise方式，老大建议可以用async/await改写，遂尝试了一下，做个记录。

原方法大致如下：

```javascript

const http = (url, method, options, data) => {

  return new Promise((resolve, reject) => {

    axios({

      url: url,

      method: method,

      headers: options,

      data: data

    })

    .then(({ data } = {}) => {

      resolve(data)

    })

    .catch((err) => {

      reject(err)

    })

  }

})

```

我们首先明确async/await默认返回的也是一个Promise对象，因此可以有下面这样的改写：

```javascript

const http = async (url ,method, options, data) => {

  try{

    let { data = {} } = await axios({

      url: url,

      method: method,

      headers: options,

      data: data

    })

    return data

  } catch (err) {

    // 异常处理

  }

}

```

这样改写代码结构更清晰，也更容易阅读。

## 改写后的错误处理机制

改写之后的错误处理机制与以前的Promise.then().catch()不同，在像上面的情况在async函数中只需要得到一次await的返回结果时，可以直接用try...catch结构截获抛出的错误，这里笔者就疑惑，如果我们返回的的promise后面加.catch会是什么情况呢？实验代码如下：

```javascript

const prom = () => {

    return new Promise(() => {

        console.log('request')

        c() // 一个不存在的函数，用来诱发错误

    }).catch ((err) => {

        console.log('err from promise')

        console.log(err)

    })

}



const test = async () => {

    try {

        let a = await prom()

    } catch (e) {

        console.log('error from try')

        console.log(e)

    }

}



test()

```

我们得到的结果是：

```
request

err from promise

ReferenceError: c is not defined

    at index.js:4

    at new Promise (<anonymous>)

    at prom (index.js:2)

    at test (index.js:13)

    at index.js:20

```

如果我们把promise.catch()去掉？则会：

```
request

error from try

ReferenceError: c is not defined

    at index.js:4

    at new Promise (<anonymous>)

    at prom (index.js:2)

    at test (index.js:10)

    at index.js:17

```

我们可以看到，在这种结构下当错误被promise截获时就不会继续向外抛，而如果promise不截获错误就会由try结构抛出错误。那我们就会想到如果我们需要多个await promise的时候直接使用try...catch结构就是一个比较好的选择，会马上抛出同步运行中第一个出错的promise，也不需要为每个promise编写catch处理错误了。

但是从另一个角度来看，我们如果需要针对每个promise的错误做出不同的处理时，就需要为每个promise加上catch。所以这两种错误处理机制的使用场景就很清楚了：如果我们对错误统一处理，使用trycatch；如果需要分别处理，我们使用promise.catch()。

参考另外一篇博客，我们可以抽象出一个方法来给每个promise加对应的处理方法，而不是疯狂写try catch结构。代码如下：

```javascript

(async () => {

 const fetchData = () => {

  return new Promise((resolve, reject) => {

   setTimeout(() => {

    resolve('fetch data is me')

   }, 1000)

  })

 }


 // 抽离成公共方法

 const awaitWrap = (promise) => {

  return promise

   .then(data => [null, data])

   .catch(err => [err, null])

 }


 const [err, data] = await awaitWrap(fetchData())

 console.log('err', err)

 console.log('data', data)

 // err null

 // data fetch data is me

})()

```

## 参考链接

[1]: https://blog.csdn.net/q3254421/article/details/88878288