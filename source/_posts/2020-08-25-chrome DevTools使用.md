---
title:      chrome DevTools全面了解使用
date:       2020-08-25
categories:
    - 知识整理
tags:
    - chrome
    - devtools
    - 调试技巧
    - 实战
excerpt:  博客自动化构建
---
# chrome DevTools全面了解使用

同样是某次面试中被问到的一些关于devTools调试技巧的问题，才发现平时自己的调试效率都太低下了，遂打算好好摸一把chrome的devTools。

## 主体

devTools如下图所示。这里我用方框标出了几大块区域，先大概介绍一下。

![QQ截图20200824213500.png](https://i.loli.net/2020/08/24/oEuTPZeqyiV6cvg.png)

这里先介绍固定的三个部分。

## 1号、5号、6号

1号区域是两个按钮，第一个表示着用鼠标在网页中pick对应的元素，点击之后会自动跳转回element部分，3号区域会将点击元素对应的html展示出来，4号区域当然也会随之变化，总而言之就是用鼠标来检查元素；第二个按钮是模拟移动设备的翻转，把页面的长和宽交换。

5号区域则是简略展示控制台报错、警告、以及一些关于网站的操作问题数量（这部分还不是很清楚，暂时没有看到可靠的资料）。

6号区域则是devTools的设置，可以找到各部分主要功能的对应设置（不详述）和其他一些更多操作，更多操作中比较常用的有调整devTools窗口的位置、搜索、运行命令或者打开当前网页的某些文件等。

![QQ截图20200824220843.png](https://i.loli.net/2020/08/24/sA1f7OzBTuUtoyM.png)

剩下的三个部分都是以2号功能栏上的功能为准而变化的，我们按照2号区域上的功能一个一个介绍。

## 功能介绍

### elements

这一块主要是检查元素。（上面的图已经是这块功能的展示我就不重复截图了）检查html架构中存在什么元素，点击对应元素的样式会在右侧4号区域展示，底部会有当前元素对应的盒子模型，顶部是一个过滤器。并且这里4号区域的样式属性是可以修改并实时反映到网页上的，也就是可以作线上调试。当然了我们看到4号区域上方不止有style，computed点击后展示的是当前元素某些样式通过计算后的结果（例如属性设置为auto或inherit的），点击某个属性会跳转回style中展示对应的属性；而eventListener展示的是一些绑定了哪些元素，出现在代码中的哪里（点击跳转source对应的代码区域）和一些设定的属性。

总的来说这一块的功能还是以检查结构和样式为主。

### console

![QQ截图20200824223505.png](https://i.loli.net/2020/08/24/LnKHTSMrDGUiwds.png)

控制台输出。相信这个区域的功能大家不会陌生，代码中的console相关操作都会在这里显示出来，消息分为三种message、warning和error。同时我们也可以在这个窗口直接输入代码运行查看结果。

我们可以再把目光放在这个区域最顶上的一栏，左边第一个按钮会在左侧弹出一个侧边栏，展示当前页面的几种信息类型，点击之后就会只显示当前类型的信息，也相当于是一个过滤器，与最右侧的default levels筛选是一样的效果。第二个禁止的按钮其实是清空当前所有的输出台输出。第三个显示top的下拉框一般不会动，大概看了一下应该是与chrome的扩展程序有关的。第四个眼睛标志用来输入live expression，一个活动的表达式，并且它会像IDE一样为你补齐关键字，笔者暂时不清楚这个的作用是什么，测试了一下输入```console.log```会不断地重复执行输出。第五个输入框就是根据关键字筛选的filter，也可以输入```url:yourUrl```来只查看来自对应url的信息。

总的来说，这部分功能还是用来查看报错信息和调试信息为主。

### sources

![QQ截图20200824230345.png](https://i.loli.net/2020/08/24/B7lAvthx8C3fYkj.png)

这是比较重要的一部分，因为这里可以查看源代码并且进行线上调试（打断点、追踪变量等）。

主要分为左、中、右三部分。左边显示当前页面所请求的资源文件，中间则是对当前选中文件的预览，这里也相当于一个编辑器，可以直接修改代码，并在行数数字左侧空白处点击即可打断点，当程序运行至断点处时，网页会出现这样的标志并且变得不可操作（蒙上了一层黑色的遮罩）：

![QQ截图20200824230942.png](https://i.loli.net/2020/08/24/xgksIFYadtXuJ3H.png)

点击右侧的箭头按钮后会继续运行至下一断点处，而跨过一点的箭头表示跳过下一个函数执行，也就是到达后一个函数执行。同样的标志我们在右侧部分的顶端也可以看到，隔壁两个带点的箭头标志分别表示到达下一个函数执行和上一个函数执行。而暂停符号是在没有断点的时候会显示出来的，可以立即暂停当前的脚本执行（比如有一些循环输出），脚本执行到断点的时候这里就会显示那个向下执行的蓝色箭头符号。

再来介绍右侧部分的下面几个框。

- 第一个是watch，用来追踪变量，点击加号输入变量名下面就会显示你所追踪的变量值，右边按钮是刷新当前变量状态，而把鼠标移到所追踪的变量上会出现一个叉，用来删除这个追踪的变量。
- 第二个scope在到达断点的时候会显示当前的几个作用域以及其中的变量，当前函数的、script中的全局变量、还有global中的变量。
- 第三个breakpoints展示打了的断点，可以在这里对打好的断点进行激活和去激活。
- 第四个是针对与XHR请求的断点，可以通过加号添加某URL，当进行对应url的请求时就会达到断点；
- 第五个DOM元素的断点；
- 第六个是全局监听者；
- 第七个是事件监听，会列好了各种事件类型，并且下拉会有具体的事件选择，勾选了之后一旦检测到对应事件发生了就会进入断点。

总的来说这个部分最重要是知道两点：如何打断点和追踪变量。更多调试技巧可以查看参考链接。

补充：在源代码中加上```debugger```也相当于打断点。

### Network

![QQ截图20200825092611.png](https://i.loli.net/2020/08/25/tSLjw3pGklM7XKF.png)

这一部分用来检测网络交互。这里从上到下分为三部分。最下面一栏，也就是主体部分，展示的是网络请求的所有文件，及其状态码、类型、发起者、大小和请求时间，点击某个文件还会展示其具体信息，如图。

![QQ截图20200825093518.png](https://i.loli.net/2020/08/25/XYFCtxeUwNcBqvp.png)

这里可以看到请求头部、入参、响应（Preview就是格式化后更美观的Response）、具体时间和cookies。

接着往上看空白的地方带有一些不同颜色的点，这一部分是对请求进行了可视化处理，依据时间轴展示了请求所占时间，这里看到是一个点是因为时间跨度被拉得比较大了，而且请求响应速度本身也确实比较快。

再往上看功能栏部分，有一个过滤器，一些其他过滤选项，以及根据文件类型过滤显示，这里我们一般在前后端联调时都选择XHR。再往上看几个按钮功能也介绍一下：红色圆圈点击后会停止对网络请求的监听，不会反映在devTools上了；禁止符号与Console一样是清空当前记录；Perserve log是指刷新页面当前的记录也不会消失；Disable cache是指打开devTools时不使用缓存；下拉框可以模拟不同网络环境，Online就是当前环境，Fast/Slow 3G顾名思义，都是比较慢的网络环境，Offline就是离线，这个工具在调试一些loading状态变更时很有用。

### Performance

![QQ截图20200825100140.png](https://i.loli.net/2020/08/25/tUJGOTVLbI3hKx2.png)

性能分析工具，需要我们主动触发才会出现如图的性能分析报告，点击录制是录制当前性能，点击右边的刷新按钮是重新载入页面并录制性能，图上的界面是已经点开了右上角的设置齿轮，可以看到这里也有几个设置，像Network部分也可以设置当前网路的环境，也可以模拟不同的CPU性能。主体部分就是用可视化来展示当前页面性能。

### Memory

![QQ截图20200825101718.png](https://i.loli.net/2020/08/25/GPJD2KaicCf3yuT.png)

内存分析工具，与上面的性能分析工具类似，也需要我们主动触发查看内存占用情况。有三种检查类型让我们选择，如图所示。

![QQ截图20200825102214.png](https://i.loli.net/2020/08/25/PDB8SmVCc5wkGM2.png)

点击下面的蓝色按钮就会根据对应模式来检测，这里以第一个模式生成快照为例，就是生成当前状态快照，可以查看当前内存中有多少个对象，占用内存的情况等。第二种模式则是根据时间轴来展示占用情况，这里可以比较方便的观察出是否有内存泄漏的情况发生。

### Application

![QQ截图20200825103102.png](https://i.loli.net/2020/08/25/dhNJY8lcAsG4zpu.png)

这一部分主要用来查看cookies，localStorage，sessionStorage等缓存的情况。

### Security

检查网站安全性。以有无https证书为基准，若不是使用https协议，这里没有信息可展示。

## 总结

把整个调试工具梳理一遍后发现还有许多地方确实是平时调试可以使用却没有注意到的，这里只把大概常用的功能捋了一遍，还有许多不同的调试技巧和功能仍然有待学习。

## 参考链接

[Chrome 调试工具【DevTools】详解](https://blog.csdn.net/github_38336924/article/details/93651496)
[Chrome DevTools — JS调试](https://segmentfault.com/a/1190000008396389)
[谷歌开发者工具调试内存 (memory面板功能使用）](https://www.jianshu.com/p/e8ac29dd230b)
[chrome调试-性能分析performance篇](https://www.jianshu.com/p/b6f87bac5381)