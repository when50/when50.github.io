---
layout: post
title: 算法的时间复杂度
date: 2020-01-30 14:44:20 +0800
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tags: [Algorithm, Time Complexity]
---

计算机算法当中的复杂度分成时间复杂度和空间复杂度，时间复杂度是指执行算法所需要的计算量，空间复杂度是指执行算法需要的内存空间。

* ***时间复杂度***

算法中基本操作重复执行的次数是问题规模n的某个函数。

* ***常见的时间复杂度***

O(1) 常数复杂度（Constant Complexity）

O(log(n)) 对数复杂度 (Logarithmic Complexity)

O(n) 线性复杂度 (Linear Complexity)

O(n<sup>2</sup>) 平方复杂度 （N Square Complexity）

O(n<sup>3</sup>) 立方复杂度 （N Cube Complexity）

O(2<sup>n</sup>) 指数复杂度 (Exponential Growth)

O(n!) 阶乘复杂度 (Factorial Complexity)

* ***时间复杂度关系***

O1 < O(log(n)) < O(n) < O(n<sup>2</sup>) < O(n<sup>3</sup>) < O(2<sup>n</sup>) < O(n!)

* ***图形描述***

![Time Complexity](https://when50.github.io/assets/img/time-complexity.jpg)



* ***推导方法***

严格的时间复杂度往往是这样的：2n<sup>2</sup> + 3n + 3，如何推导为标准的时间复杂度呢？方法如下：

1. 用1取代所有加法常数
2. 只保留最高阶项
3. 如果最高阶项存在且不是1，则去除掉这个项相乘的常数





