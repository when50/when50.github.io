---
layout: post
title: ReSwift Tutorial
date: 2020-01-15 14:28:20 +0800
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tags: [ReSwift, unidirectional data flow, design pattern, Swift]
---

###ReSwift教程：记忆力游戏

[原文：ReSwift Tutorial: Memory Game App][2]

随着iOS app数量的增长，MVC模式像“goto”一样逐渐丧失了首选地位。

有了更多可伸缩的架构模式供iOS开发者选择，例如MVVM，VIPER，还有Riblets。看起来这些架构迥然不同，但它们有个相同的目标：把多项数据流拆分成单一职责的代码块。在多向数据流中，不同模块之间数据流向多个方向。

有时候，你不想（需要）多向数据流，换而言之，你想要数据流是单向的，即单向数据流（*unidirectional*）。

ReSwift是啥？

ReSwift是帮你创建类Redux架构的Swift小框架。

ReSwift由4个主要部分组成：

* Views：响应Store变化，把其展示在屏幕上。发送Actions。
* Actions：在app中发起状态改变。Reducer负责处理Action。
* Reducers：直接修改存储在Store中的应用状态。
* Store：存储应用当前的状态。Views之类的模块可以订阅、响应Store的改变。

ReSwift展现了许多有趣的有点：

* 超强的约束：把逻辑并不真正相关的小块代码放在一个方便的位置，是非常有诱惑力的。ReSwift严格约束了（行为）在哪里发生和发生了什么（行为）。
* 单向数据流：多向数据流开发的App很难理解和调试。一个改变会导致程序发送一连串数据的事件。单向数据流更容易设想变化，并且可以大大减少代码的理解成本。
* 便于测试：大部分（业务）逻辑包含在函数式的Reducers当中。
* 平台无关：ReSwift所有组件都是平台无关的。你可以方便的应用到iOS、macOS或者tvOS当中。

对比多向和单向数据流

举个例子来说明前面的数据流。VIPER框架支持组件之间的多向数据流：

![img](https://koenig-media.raywenderlich.com/uploads/2017/02/first-480x239.png)

对照ReSwift框架的单向数据流：

![img](https://koenig-media.raywenderlich.com/uploads/2017/02/Image-2017-02-26-at-11.21.46-PM-480x196.png)

数据只能单向流动，所以在app中单向数据流展示数据和追踪问题容易得多。

入门

下载[基础工程][1],其中包含了骨干代码和框架，还包含了ReSwift。

首先，你需要装配ReSwift。你要开始装配应用程序的核心：状态。

打开AppState.swift，创建AppState结构体，实现了StateType协议。

```swift
import ReSwift
struct AppState: StateType {
  
}
```

这个结构体定义了整个app的状态。

在创建包含AppState的State之前，必须要先创建主Reducer。

![img](https://koenig-media.raywenderlich.com/uploads/2017/03/reducer-480x196.png)

Reducers只是用来直接改变Store持有的AppState当前值的代码块。只有Actions可以发起Reducer，从而改变应用程序的当前状态。Reducer根据Action的类型改修改AppState的当前值。

!!!A



[1]: https://koenig-media.raywenderlich.com/uploads/2017/07/MemoryTunesStarterApp_starter.zip	"Base Project"
[2]: https://www.raywenderlich.com/516-reswift-tutorial-memory-game-app "原文"





