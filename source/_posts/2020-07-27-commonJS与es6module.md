---
title:      commonjs, es6 module知识扫盲
date:       2020-07-27
categories:
    - 知识整理
tags:
    - 前端
    - JavaScript
    - commonjs
    - es6
    - es6 module
    - amd
    - cmd
excerpt: import？require？
---
# commonjs, es6 module知识扫盲

## 前言

对于要了解的技术惯例三问：是什么，为什么要使用，要怎样使用。从最接近开发层面来讲，也是笔者当初最早接触到这两者不同的是require和import引入模块的不同语法。也就是说其实这两种规范是为了方便我们进行**模块化编程**的，无论是从网络上获取包还是自己封装复用的模块，我们都需要用这两者的其中一种来调用。（当然我们现在多数还是使用import，也就是es6 module规范）js是一种依托于浏览器环境来运行的脚本语言，现在我们知道我们在开发的时候可以像python等其他编程语言调用很多现成的包去使用，要做到这个实际上需要我们js有自己的一套环境去运行，支持这一点的就是后来出现的nodejs，也正因为是有了nodejs才让我们js的模块化编程成为了可能。而写过nodejs的人都知道，nodejs里调用包的时候使用的是require也就是它一直遵循的都是commonjs规范。我们其实可以把commonjs看成es5时期的模块化管理规范，实际上我们在使用es6 module（import）的时候babel也会把它转化成commonjs（require）。进一步去思考模块化为什么需要这些规范，我们都知道模块化很方便很好用，但实际上它除了要引入文件（文件依赖）以外，还要解决命名冲突的问题。下面我们先从语法入手看看他们的区别，并用概念区别去解释他们为什么有这样的语法区别。

## 语法区别

### CommonJS

```javascript
// 导出
module.exports = {  a: 'a',  b: 'bb',}
// 引入
var mod = require('/src/mod.js')
console.log(mod.a)
```

这里顺带提一下，我们在正式导出时都应该是用module.exports而非exports, exports只是module.exports的一个引用，最后require导入的是module.exports的内容。

### ES6 module

```javascript
// 导出
export function func() => {}
export const a = 'a'
// 按需引入
import { func } from '/src/mod.js'
func()
// 默认导出
export default {b: 'bb'}
// 默认引入
import mod from '/src/mod.js'
console.log(mod.b)
```

解释一下，普通导出是对应按需引入来使用的，普通导出是直接导出一个变量，在引入了它的文件中可以直接使用这个变量，引入的时候也用大括号来写清楚引入的变量；而默认导出则导出一个匿名的对象，在导入的时候需要定义一个变量名，使用的时候导出的东西就在这个变量属性下，可以看作是类似commonJS的实现。

### 区别？

- 从语法我们不难看出来，commonJS导入方式更像是创建一个变量，把这个引入文件的内容放进去然后使用，而es6 module（后文简称为ESM）更接近于我们语义上的直接引入使用。
- commonJS导入实际上是对引入模块的一种拷贝，导入之后的内容我们是可以修改的；而ESM则只是一种只读引入。
- 而也正由于这种拷贝特性，并且require引入后再次require是会直接引用上一次缓存里的值，因此是一种静态引入，而import引入则是动态的，原模块改变的时候引入的部分也会随之改变。
- 其实语法上还有另一个区别，import是在编译时调用导入的所以必须放在文件开头，而require是执行时才调用导入，所以理论上可以放在文件中的任何位置。对于导出来说，commonjs的module.export是同步的，而ESM的导出则是异步的并且必然是模块内的操作都执行完了之后才会导出指定变量。
- require在引入的时候就会执行一次模块代码，import只是一个引用标志，等执行到引入部分代码的时候才根据这个引用标志去找那部分只读引入。

回到我们在前言中提过的，文件依赖问题我们当然是解决了，而命名空间的问题也很简单，令引入的模块有单独的作用域即可。而这两种方法当然都把内部变量放在了模块内部的，有着各自的命名空间，我们只需要在导入的时候不要命名重名模块或者给予别名就可以了。

## JS模块化的发展

上面我们谈了在日常写代码中碰到的比较多的两种模块化编程方式，这里我们来整体捋一下模块化的编程思想是怎么发展过来，并且把这些概念再做一个对比。按时间线：CommonJS（NodeJS） → AMD（Require.js） → CMD（Sea.js） → UMD（AMD + CommonJS） → ESMCommonJS和ESM我们上面都介绍过了，稍微说说中间三个。

### AMD

AMD，Asynchronous Module Defination，异步模块定义，实际上就是在CommonJS的基础上加上一个define方法，这个方法有两个传入参数，第一个是一个数组，表示导入模块的路径，第二个往后就是按照这个数组导入顺序的回调函数，从而支持导入异步回调。使用AMD主要是为了优先照顾浏览器的环境，我们都知道commonJS是跑在nodejs里的。AMD只是一种规范，实际上我们要使用Require.js工具库来实现它。这里也不详述了。

### CMD

CMD，Common Module Defination，公共模块定义，一个文件就是一个模块，全局的define函数来加载依赖。相对的Sea.js就是一个实现CMD的工具库。

### AMD和CMD区别？

AMD和CMD的加载都是异步的，所以加载时机是相同的。只是执行时机不相同，AMD在加载完模块之后，就会执行一遍并调用回调；而CMD加载完之后先不执行，等代码运行到要使用相关模块时才去执行。我们可以看到AMD思想跟CommonJS是比较接近的，CMD是跟ESM比较接近的。

### UMD

UMD，Universal Module Defination，统一模块定义，实际上就是用AMD的语法包裹CommonJS的语法，并用IIFE（立即执行函数）包括起来整个操作，这里也不详细贴代码说了。

## 参考链接

[CommonJS模块与ES6模块的区别](https://www.cnblogs.com/unclekeith/archive/2017/10/17/7679503.html)
[require和import的区别是什么？看这个你就懂了](https://segmentfault.com/a/1190000014434944)
[理解CommonJs、AMD、CMD、ES6模块](https://www.jianshu.com/p/67ce52c93392)
[很全很全的JavaScript的模块讲解](https://segmentfault.com/a/1190000012464333?utm_source=sf-related)
