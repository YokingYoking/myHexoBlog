---
title:      vue-loader官方文档读后感
date:       2020-07-01
categories:
    - 知识整理
tags:
    - 前端
    - vue
    - vue-loader
    - vue-cli
excerpt: vue-loader
---
# vue-loader官方文档读后感

最近接触到了vue-loader，结合实际开发中的一些使用，记录一些心得体会。

## 上手

vue-loader在我们使用vue-cli创建项目的时候会自动安装上，而实际上安装这个插件的时候是通过webpack.config.js来配置的，顺带一提vue-cli3已经高度集成了webpack，所以我们直接在vue.config.js里面修改配置。按照官方文档提供的webpack手动设置如下：

```javascript

// 配置前请先用npm安装好vue-loader，sass-loader，node-sass（以使用预处理器）

// webpack.config.js

const VueLoaderPlugin = require('vue-loader/lib/plugin')



module.exports = {

  mode: 'development',

  module: {

    rules: [

      {

        test: /\.vue$/,

        loader: 'vue-loader'

      },

      // 它会应用到普通的 `.js` 文件

      // 以及 `.vue` 文件中的 `<script>` 块

      {

        test: /\.js$/,

        loader: 'babel-loader'

      },

      // 它会应用到普通的 `.css` 文件

      // 以及 `.vue` 文件中的 `<style>` 块

      {

        test: /\.css$/,

        use: [

          'vue-style-loader',

          'css-loader',

          'sass-loader', // 使用预处理器

        ]

      }

    ]

  },

  plugins: [

    // 请确保引入这个插件来施展魔法

    new VueLoaderPlugin()

  ]

}

```

当然了如上面所说的，我们直接使用vue-cli创建的项目不需要这么去做了。详细的vue.config.js配置可以去看vue-cli的官方文档来了解。我们去查看sass-loader的官方文档，它们都是通过webpack.config.js来配置的，不过我们要清楚vue-loader和sass-loader是两个独立的插件，只是这里示例直接写在一起了，始终要明白vue-loader的作用只是为了解析vue语法并且提供一些实用功能（比如热重载）。

## 功能了解

我们上面提过了，vue-loader和其他loader一样，顾名思义用于加载vue文件，解析其语法，而标准的vue文件里有三个块\<template>,\<script>,\<style>。在实际应用中用得比较多与vue-loader相关的是style块，lang指定解析指定的后缀名文件，scoped表示style块只影响这个子组件（scoped解析相关问题可以见[另一篇博客](http://www.yokingcloud.cn/2020/06/30/2020-06-30-vue%E7%9A%84scoped%E6%A0%B7%E5%BC%8F%E9%97%AE%E9%A2%98/)），这两项都是由vue-loader去解析的。下面我们来看看如何自定义一个与这三个标准块同一层级的块，并在vue-loader中配置解析。

### 自定义块

依旧是看官方文档的例子：

> 在 .vue 文件中，你可以自定义语言块。应用于一个自定义块的 loader 是基于这个块的 lang 特性、块的标签名以及你的 webpack 配置进行匹配的。<br>
> 如果指定了一个 lang 特性，则这个自定义块将会作为一个带有该 lang 扩展名的文件进行匹配。<br>
> 你也可以使用 resourceQuery 来为一个没有 lang 的自定义块匹配一条规则。

这么看不直观，来看个官网的例子。比如说我们想自定义一个\<doc>块，我们就需要在webpack.config.js里写好配置：

```javascript

module.exports = {

  module: {

    rules: [

      {

        resourceQuery: /blockType=docs/,

        loader: require.resolve('./docs-loader.js')

      }

    ]

  }

}

```

这里的docs-loader就是自己写好的一个自定义loader，这里就没有规定具体怎么写了，感兴趣可以去看webpack上各类常用loader的源码。配置好之后我们直接在.vue中使用\<doc>块即可。

### 热重载

顾名思义，在我们开发中对文件进行修改，效果会马上反应到我们的项目上。它是默认打开，并且在vue-cli中开箱即用，具体不用了解得很清楚，我们只需要知道在需要的时候如何配置关闭即可：

```javascript
module: {

  rules: [

    {

      test: /\.vue$/,

      loader: 'vue-loader',

      options: {

        hotReload: false // 关闭热重载

      }

    }

  ]

}

```

## CSS Modules

vue-loader官网对CSS Modules的介绍如下：

> CSS Modules 是一个流行的，用于模块化和组合 CSS 的系统。vue-loader 提供了与 CSS Modules 的一流集成，可以作为模拟 scoped CSS 的替代方案。

笔者认为直观理解，就是通过最简单的方式（不需要另外使用一种语言如sass），把css当作js模块导入，从而把css巧妙地转变成一种编程语言（我们都知道css其实算不上是一种编程语言），并且通过自动编译过程中把类名写成唯一的哈希名（并且这个哈希名生成规则可以在webpack中配置）从而达到scoped css的效果。而vue-loader也定义了一定的规则去使用css modules：

首先要在css-loader的配置中打开：

```javascript

{

  module: {

    rules: [

      // ... 其它规则省略

      {

        test: /\.css$/,

        use: [

          'vue-style-loader',

          {

            loader: 'css-loader',

            options: {

              // 开启 CSS Modules

              modules: true,

              // 自定义生成的类名

              localIdentName: '[local]_[hash:base64:8]'

            }

          }

        ]

      }

    ]

  }

}

```

按照官方文档的指引来使用：

> 在你的 \<style> 上添加 module 特性：
>
> ```css
> <style module>
> .red {
>   color: red;
> }
> .bold {
>   font-weight: bold;
> }
> </style>
> ```

> 这个 module 特性指引 Vue Loader 作为名为 $style 的计算属性，向组件注入 CSS Modules 局部对象。然后你就可以在模板中通过一个动态类绑定来使用它了：<br>
>
> ```html
> <template>
>  <p :class="$style.red">
>    This should be red
>  </p>
></template>
> ```
>
> 因为这是一个计算属性，所以它也支持 :class 的对象/数组语法：<br>
>
> ```html
> <template>
>  <div>
>    <p :class="{ [$style.red]: isRed }">
>      Am I red?
>    </p>
>    <p :class="[$style.red, $style.bold]">
>      Red and bold
>    </p>
>  </div>
></template>
> ```
>
> 你也可以通过 JavaScript 访问到它：<br>
> ```javascript
><script>
>export default {
>  created () {
>    console.log(this.$style.red)
>    // -> "red_1VyoJ-uZ"
>    // 一个基于文件名和类名生成的标识符
>  }
>}
></script>
> ```

这就是css modules在vue中的基本用法，继续深入了解可以查看[官方文档](https://github.com/css-modules/css-modules)，参考链接中也附上了阮一峰大神的日志。

### 代码校验（Linting）/ 提取css / 测试

实际上像上面提到的sass-loader一样，我们使用的es-lint，stylelint，mini-css-extract-plugin都是独立的插件，也是一同配置在webpack中即可，也不会涉及到复杂的使用。具体就不在这里详述，可以去查看相关文档。

## 与vue.config.js对接

上面我们提到的都是vue-loader在webpack中的配置，但正如上面所说的，vue-cli项目中的配置都在vue.config.js中，需要具体配置的时候当然是要去查[官方的配置文档](https://cli.vuejs.org/zh/config/#css-loaderoptions)，这里简单介绍几个本人在业务中接触过的，感觉用的应该还会挺多的几个配置项：

### configureWebpack 与 chainWebpack

configureWebpack是简单的webpack配置方法：

```javascript

configureWebpack: { // 传入对象

    plugins: [

      new MyAwesomeWebpackPlugin()

    ]

  }


configureWebpack: config => { // 传入方法

    if (process.env.NODE_ENV === 'production') {

      // 为生产环境修改配置...

    } else {

      // 为开发环境修改配置...

    }

  }

```

关于chainWebpack在vue-cli官方文档中有介绍：

> Vue CLI 内部的 webpack 配置是通过 webpack-chain 维护的。这个库提供了一个 webpack 原始配置的上层抽象，使其可以定义具名的 loader 规则和具名插件，并有机会在后期进入这些规则并对它们的选项进行修改。它允许我们更细粒度的控制其内部配置。

理解一下就是通过一个链式属性修改来定义好config，最常用的就是修改alias中的路径别名了：

```javascript

const path = require('path');



chainWebpack: (config) => {

    // 设置 alias

    config.resolve.alias.store

      .set('@$', path.resolve(__dirname, 'src'))

      .set('@images', path.resolve(__dirname, 'src/assets/images'))

      .set('@styles', path.resolve(__dirname, `src/theme/${theme}`))

  },

```

### css.loaderOptions

这个就跟我们开篇的主题息息相关了，配置样式的loader，如下：

```javascript

css: {

    loaderOptions: {

      // 给 sass-loader 传递选项

      sass: {

        // @/ 是 src/ 的别名

        // 所以这里假设你有 `src/variables.sass` 这个文件

        // 注意：在 sass-loader v7 中，这个选项名是 "data"

        prependData: `@import "~@/variables.sass"`

      },

      // 默认情况下 `sass` 选项会同时对 `sass` 和 `scss` 语法同时生效

      // 因为 `scss` 语法在内部也是由 sass-loader 处理的

      // 但是在配置 `data` 选项的时候

      // `scss` 语法会要求语句结尾必须有分号，`sass` 则要求必须没有分号

      // 在这种情况下，我们可以使用 `scss` 选项，对 `scss` 语法进行单独配置

      scss: {

        prependData: `@import "~@/variables.scss";`

      },

      // 给 less-loader 传递 Less.js 相关选项

      less:{

        // http://lesscss.org/usage/#less-options-strict-units `Global Variables`

        // `primary` is global variables fields name

        globalVars: {

          primary: '#fff'

        }

      }

    }

  }

```

这里只配置了loaderOptions，css还有其他的属性配置也会写在css下。

### devServer

这个我们用的最多的就是配置代理了，尤其是处理跨域请求的时候：

```javascript

devServer: {

    proxy: {

      '/api': {

        target: '<url>', // 目标服务器url

        ws: true,

        changeOrigin: true

      },

      '/foo': {

        target: '<other_url>'

      }

    }

  }

```

### 文件路径配置

这些配置有publicPath（部署应用包时的基本 URL），assetPath（放置生成的静态资源的目录），outputDir（生成的生产环境构建文件的目录），indexPath（指定生成的 index.html 的输出路径 (相对于 outputDir)）等，就是根据配置名理解即可。

## 总结

这篇文章的整体知识比较繁杂，并不十分成体系，我们去理解的话就是vue-loader作为webpack插件解析vue语法，所以从底层配置它需要webpack.cofing.js；而vue-cli是一个脚手架工具，它高度集成了很多模块，包括webpack和vue-loader，我们需要通过一个vue.config.js来配置底部的一些插件，比如vue-loader。

## 参考链接

[vue-loader官方文档](https://vue-loader.vuejs.org/zh/)<br>

[vue-loader官方文档spec](https://vue-loader.vuejs.org/zh/spec.html )<br>

[阮一峰的日志](http://www.ruanyifeng.com/blog/2016/06/css_modules.html )<br>

[css module入门](https://segmentfault.com/a/1190000014722978 )

