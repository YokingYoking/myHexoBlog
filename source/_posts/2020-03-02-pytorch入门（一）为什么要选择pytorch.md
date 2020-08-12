---
title:      pytorch入门（一）为什么要选择pytorch
date:       2020-03-02
categories:
    - 学习笔记
tags:
    - 深度学习
    - pytorch
    - python
excerpt: 从pytorch入门人工智能。
---

# 选择pytorch的好处有什么

1. python化的编程风格（相对于tensorflow的混合编程风格而言）
2. tensor与numpy相同的格式，但计算可以放到gpu上进行加速计算
3. 可以生成一个动态计算图Dynamic Computation Graph(最主要特性)

## 关于动态计算图

圆形节点表示一种运算如MM，Add
方形节点表示变量
箭头表示依赖关系
运算顺序从上到下，相当于多个函数嵌套 => 计算模型更加灵活复杂

也能让BP算法随时进行

![Dynamic Computation Graph](https://i.loli.net/2020/03/02/JioUhPey1rZHM75.png)

## DCG的实际编程实现

```python
import torch
x = torch.ones(2, 2, requires_grad=True) # 需要打开requires_grad，定义一个自动微分变量
print(x)
y = x + 2
print(y.grad_fn)  # 查看y在动态计算图图中的父节点
z = torch.mean(y*y)  # 对y矩阵的值求平均值
print(z)
print(z.data)
z.backward()  # 求叶节点梯度计算的结果存到x.grad中 => 自动微分
print(x.grad)
```

![上述代码的dcg](https://i.loli.net/2020/03/02/N9xtZV75TR6kmri.png)

结果如图

![result](https://i.loli.net/2020/03/02/hBuElwUs8CDWPrO.png)

## 总结

- BP算法是深度学习网络实现的一个重要基础，所以pytorch中的dcg可以让bp算法随时进行这个特点是我们选择它的一个最重要因素
- 其次通过.cuda可以把tensor放到gpu上进行运算加速
- 再次就是其统一而简洁的python化编程风格