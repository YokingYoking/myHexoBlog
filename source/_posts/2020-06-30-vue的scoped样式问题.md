---
title:      vue的scoped样式问题
date:       2020-06-30
categories:
    - 知识整理
tags:
    - 前端
    - vue
    - 框架
    - scoped
    - css
    - vue-loader
excerpt: 与vue-loader有关
---
# vue的scoped样式问题

在日常修bug中忽然发现自己修改好的样式竟然没有正式生效，感到十分奇怪，查阅一些资料之后发现与vue-loader的scoped css有关，在这里做一个记录并深入了解一下背后的原因。

## scoped css

我们在开发过程中在组建中使用scoped的原因就是限定样式的生效范围，只在本组件生效，以免污染全局样式。而这个功能是由vue-loader提供的，我们来看一下官方文档怎么说：

> 使用 scoped 后，父组件的样式将不会渗透到子组件中。不过一个子组件的根节点会同时受其父组件的 scoped CSS 和子组件的 scoped CSS 的影响。这样设计是为了让父组件可以从布局的角度出发，调整其子组件根元素的样式。

看到这里问题就很明显了，由于项目中使用的是element-ui，所以下面使用的一些封装好的组件自然会使用scoped，因此我们在自己的页面组件中修改它的样式也自然不会成功生效修改。

## 解决方法

这里官方文档也给出了解决方案：

> 如果你希望 scoped 样式中的一个选择器能够作用得“更深”，例如影响子组件，你可以使用 >>> 操作符：<br>
> ```html
> <style scoped>
> .a >>> .b { /* ... */ }
> </style>
> ```
> 上述代码将会编译成：<br>
> ```css
> .a[data-v-f3f3eg9] .b { /* ... */ }
> ```
> 有些像 Sass 之类的预处理器无法正确解析 >>>。这种情况下你可以使用 /deep/ 或 ::v-deep 操作符取而代之——两者都是 >>> 的别名，同样可以正常工作。

这样做就可以使用scoped来避免全局样式被污染的同时，对于某些我们需要的特定样式进行深度修改。

## 参考链接

[1]: https://vue-loader.vuejs.org/zh/guide/scoped-css.html#%E6%B7%B1%E5%BA%A6%E4%BD%9C%E7%94%A8%E9%80%89%E6%8B%A9%E5%99%A8
