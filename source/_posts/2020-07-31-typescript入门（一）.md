---
title:      Typescript入门（一）
date:       2020-07-31
categories:
    - 知识整理
tags:
    - 前端
    - TypeScript
    - 入门
excerpt:  Typescript入门（一）
---
# Typescript入门（一）
看网上都说ts很香，看身边同事也很多在用，遂入坑。
## 安装
没什么难度：

```bash
npm install -g typescript
```

ts需要用编译器（当然了开发中用IDE插件和自带的就可以了）（tsc=ts compile）

```bash
tsc helloWorld.ts
```

## 粗读文档

大概过了一遍中文版的入门教程（个人觉得也可以看作中文文档吧），感触比较深的就是ts把js从脚本语言变成了一种强类型编译语言（不然为什么会叫type呢），并更加向真正的面向对象靠拢了，又因为笔者学过一丢丢的java，所以有点既视感是把java的一些东西融合到这里面去了。下面针对一些自己觉得要注意的区别点对ts入门做个记录。

### 类型定义及扩展

#### 基本类型

所有人都知道ts相对于js最核心的改变就是变成了强类型语言，在编译过程中对定义类型进行了检查。与js一致的，六种基本数据类型：**number,string,boolean,Object,null,undefined** (ES6还有新增的symbol)，与其他语言稍微不同的是对变量定义的类型会现在变量名的后面，而对于函数则是把返回值类型写在大括号的前面：

```typescript
let num: number = 1
function func(args: boolean): string {  return args ? 'true' : 'false'}
```

```java
// java
String str = "string"
```

另外ts引入了其他语言很常见的void类型（空），void类型只能被赋值为**null**或**undifined**，它常用在没有返回值的函数定义中。定义变量的时候也会出现它可以使多种类型的情况（这个我们也称作联合类型），这个时候用或运算符（|）来补充即可（string | number），但是对于未确定的变量，只能使用共同的属性。同时如果不能确定是什么类型，还可以用any表示，或者不给变量指定类型，编译器也会直接把这个变量当成any。但是在不指定类型为这个变量赋值之后，这个变量的类型就会指定为这个值的类型，在后续如果想修改为别的类型是会报错的。

```typescript
let name = 'Jacky'name = 1 // 会报错
let myName: any = 'Sting'myName = 2 // 不会报错
let sth: string | numberconsole.log(sth.length) // 会报错
```

#### 对象/数组/函数类型

我们都知道除了这些基本类型，还有Object下还有些类型细分（也就是typeof无法辨别，需要用instanceof的部分），也就是对象/数组/函数等。为了定义对象的类型，我们引入一个面向对象的概念：接口，来对对象进行描述，有了这个“描述”，就相当于定义了这个对象的类型。接口在面向对象中原本指的是对对象行为的一种抽象，这个我们在后面会谈到。在现在这个应用场景下，它的意义更多是描述这个对象的“形状”。来看代码：

```typescript
interface Person { // 可以理解为人都必有属性就是名字和年龄，也就是Person的“形状”  
    name: string,
    age: number,
}
let jacky: Person { // 创建了一个具体的Person类型变量，它必须包含我们定义的Person应该有的属性  
    name: 'Jacky',
    age: 22,
    }
```

在上面的代码我们演示了接口的用法，可以看到有一点通过类创建实例的感觉。只不过类型赋值是严格的，我们这个变量与接口“形状”完全一致，这些属性不能多也不能少。但是当然了像上面可选类型一样，不仅我们接口里的属性也是可选的，并且可以定义一个任意属性：

```typescript
interface Person {  
    age: number,
    name?: string, // 定义该类型变量可以没有这个属性（增加下限）  
    [propName:string]: any, // 定义该类型变量的时候可以另外加别的任意属性（增加上限）
}
```

但这里要注意：
1. 每个接口的任意属性只能有一个；
2. 其它定义好的属性的类型必须是这个任意属性的子类型，比如demo中的any就是number和string的父类，又或者我们写成```number|string```也是可以的。
另外接口还可以加限制属性只读，只需要在属性名前加一个readonly即可。

数组的定义方法也很简单：```let arr:number[] = [1,2,3]```用接口来表示也是可以的（理解数组的本质就是键名为从0开始的数字的对象）：

```typescript
interface IArray {  [index:number]: number,}
```

说到这里，我们在原本js中还接触过类数组（array-like object），类数组是无法直接用数组的定义方式去定义的，只能通过接口的方式来定义其类似数组的形状，这里以内置的IArgument来举个例子：

```typescript
interface IArguments {
    [index: number]: any,
    length: number,
    callee: Function,
}
```

接下来来看看函数，其实很多地方都比较相同，我们来看一个例子就清晰了：

```typescript
let mySum: (x: number, y: number = 1, z?:number, ...rest) => number = function (x: number, y: number): number {
    let sum = x + y + z
    for(let i = 0;i < rest.length;i++) {
       sum = sum + rest[i]
    }
    return sum
};
```

1. 普通的定义函数，定义有的参数是不能缺少的
2. y定义的是默认参数，z是可选参数，rest是剩余参数
3. 箭头符号=>用以定义函数，可以把箭头的左半边看作一个接口，右边是具体定义

这里顺便说一下接口如何定义函数：```interface func {(x: string, y: string): string;}```，注意就是规定好参数类型和返回值类型。ts中的函数还多了一个很重要的功能：重载，在别的编程语言中我们知道，重载是对函数重复定义从而允许一个函数接受不同数量或类型的参数时，作出不同的处理。这里直接引用中文入门教程里的例子：
> 比如，我们需要实现一个函数 reverse，输入数字 123 的时候，输出反转的数字 321，输入字符串 'hello' 的时候，输出反转的字符串 'olleh'。
> 利用联合类型，我们可以这么实现：
> 
> ```typescript
>function reverse(x: number | string): number | string {
>   if (typeof x === 'number') {
>        return Number(x.toString().split('').reverse().join(''));
>    } else if (typeof x === 'string') {
>        return x.split('').reverse().join('');>    }
>}
>```
>
>然而这样有一个缺点，就是不能够精确的表达，输入为数字的时候，输出也应该为数字，输入为字符串的时候，输出也应该为字符串。这时，我们可以使用重载定义多个 reverse 的函数类型：
>
>```typescript
>function reverse(x: number): number;
>function reverse(x: string): string;
>function reverse(x: number | string): number | string {
>    if (typeof x === 'number') {
>        return Number(x.toString().split('').reverse().join(''));
>    } else if (typeof x === 'string') {
>        return x.split('').reverse().join('');>    }
>}
>```
>
>上例中，我们重复定义了多次函数 reverse，前几次都是函数定义，最后一次是函数实现。在编辑器的代码提示中，可以正确的看到前两个提示。注意，TypeScript 会优先从最前面的函数定义开始匹配，所以多个函数定义如果有包含关系，需要优先把精确的定义写在前面。

#### 泛型
泛型也是一个十分重要的知识点，笔者最早接触到这个概念是在java中，泛型就是我们在不确定这个类型是什么但是它与两个以上的变量有关联的时候（比如不确定输入参数，但是知道输出结果与输入参数类型一致），我们用一个变量来表示这个类型，无论是接口/类还是函数/对象，都可以使用，下面同样只从教程引用一个简单的例子，举一反三使用即可：

>```typescript
>interface CreateArrayFunc<T> {
>    (length: number, value: T): Array<T>;
>}
>
>let createArray: CreateArrayFunc<any>;
>createArray = function<T>(length: number, value: T): Array<T> {
>    let result: T[] = [];
>    for (let i = 0; i < length; i++) {
>        result[i] = value;>    }
>    return result;
>}
>
>createArray(3, 'x'); // ['x', 'x', 'x']
>```

补充：
1. 泛型可以有多个，一般用T，U表示；
2. 还可以对泛型进行限制，让反省继承一个接口：\<T extends anInterface\>。

## 总结

入门ts的系列第一篇文章，先只介绍一下ts比较核心的type概念，剩下的后面继续介绍。

## 参考链接

[ts中文入门教程](https://ts.xcatliu.com/)
[ts官方文档](https://www.typescriptlang.org/docs/handbook/basic-types.html)
