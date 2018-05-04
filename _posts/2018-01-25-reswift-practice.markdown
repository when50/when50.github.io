---
layout: post
title: ReSwift最佳实践
date: 2018-01-25 16:38:20 +0800
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tags: [单向数据流, ReSwift, 最佳实践]
---
&emsp;&emsp;接触单向数据流是看了喵神的“[单向数据流动的函数式 View Controller]”一文,觉得这个架构太酷了，随即科普（搜索）了下，就和[ReSwift]开始了第一次亲密接触，因为觉得思想和框架比较简单、组件代码赞也比较多，就立即就开始了实践，后来为轻率付出了不大不小的代价。哗哗哗，开始撸代码，搞定收工。哔哔哔！QA的同学反馈用了ReSwift的几个界面都有内存泄露。认真负责的把代码看了好几遍，觉得不是我代码的问题，好在泄露并不严重没有什么可怕的后果，又到了交付时间点只好带着先上了。

&emsp;&emsp;后面的需求一波接一波，上线后也确实没有增加用户崩溃数，所以这个事就先放下了，但是如鲠在喉的感觉...终于熬到了这个版本，手上的需求不多很快就开发完了，可以腾出手来解决可恶的问题了。翻了无数论坛，似乎大家并没有遇到这个问题，只好去参考Demo，突然发现自己犯了一个初学者的错误，在每个View Controller对象当中都持有了一个Store和一个State，这只能出现在Demo当中的代码被我放倒项目中了！这里其实需要说明一下：对于同一个界面，不管是列表页、详情页等等，都可以对应到一个独一无二的State上，因为只有用户可以看到和交互的界面才会订阅Store当中的State，试想一下每个详情页，本来布局相同，仅仅是数据不同，如果都对应到不同的Store和State那就不存在重用了！并且还有一点，State和Store的存在应该完全和View Controller无关，因为State和Store才是业务，View Controller仅仅是显示数据并接受用户交互。这里的内存泄露黑手也就浮现出来了：View Controller -> Store -> Subscriber(即View Controller)。

&emsp;&emsp;理清了问题解决就有办法了，把Store和State从View Controller中独立出来，每个界面在显示前初始化（恢复：界面不是刚刚创建就需要恢复）State，并订阅Store。其实我觉得ReSwift应该抽象一层State的serialization API，但是并没有，所以只能自行恢复State了，我猜这也是为啥Store当中的State竟然没有封装的原因吧：P。

&emsp;&emsp;如果Bug到这里就彻底修复了就没啥戏剧性了是吧？做了上面的调整之后我发现泄露和循环引用还是存在，我到底做错了什么？还是ReSwift的锅呢？到这时候我还是没有怀疑大神的代码（其实我并不知道作者是不是大神，就是觉得在理论上做出来了就应该算吧，后来我读框架源码的时候确实觉得作者水平很高）。一直到我搜索到了一个commit[Fix retain cycle in SubscriptionBox]，觉得大神开始接地气了，呵呵呵。然后更新了新版本，再然后就清爽了。

PS：不太善于码字，本来还想展开讲讲ReSwift源码的，因为只看了一部分，等之后有机会的吧。手好累啊：）

总结一下：

 * 需要认真体会每种框架的设计思想，动手实践，最好别轻易在项目中使用，如果像我一样懒并且有一帮可靠的QA的话，可以尝试下（no zuo no die）
 * 大神也是人，也会犯错，尽管问题往往是我们自己导致。不妨尝试读一读组件的源码，肯定会有收获，不是说要质疑大神，学习一下也是不错的


[单向数据流动的函数式 View Controller]: https://onevcat.com/2017/07/state-based-viewcontroller/
[ReSwift]: https://github.com/ReSwift/ReSwift
[Fix retain cycle in SubscriptionBox]: https://github.com/ReSwift/ReSwift/pull/278/files/b0097a5aa41f8329b382c649163ddfb58b83c958
