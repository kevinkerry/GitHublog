title: Study iOS - MVC
date: 2015-01-05 19:38:55
categories: 开发
tags: [iOS, MVC]

---

iOS学习起步阶段，抽空看完了 Stanford 老头的 Developing Applications for iOS 系列视频，挺不错的，浅显易懂。打算把“观后感”做成笔记，挑些重要的点，理一理，省得以后还要重新看整个教程的文档。

### Developing Applications for iOS 观后感系列：MVC 设计模式

*相关视频章节：1*

---

<!--more-->

先上图！

![MVC](/img/Study_iOS_MVC/1.0.MVC.png)

这幅图描述的是单一的 **`MVC`** 模式。普通的 iOS 开发就靠这个 MVC 模式走遍天下了。（有牛人开创了个 [ReactiveCocoa 模式] [1]）

#### 理解一下 MVC 模式：

1. V 和 M 永远也不能在一起，无法做直接通信。C -> M 存在单向的直接通信。C -> V 存在单向的直接通信。也就是说，C 永远是老大，它负责吩咐 V 和 M 办事，而且不受 V 和 M 的直接影响。
2. 通过将 V 设置成 C 的 *`outlet`* *(Control Drag)*，可以实现 C 对 V 的直接通信，即 C -> V。而 V --> C 的间接通信方式包括2种：*`target-action`* 以及 *`delegate（data source 也可以理解成 delegate）`*。
3. C -> M 是理所当然的。而 M --> C 的间接通信是通过 *`Notification & KVO`* 实现的。

基于单个的 MVC，我们可以构建复杂的多 MVC。

![MVCs](/img/Study_iOS_MVC/1.1.MVCs.png)

**但不管多复杂，总是坚持这个原则**：一个 MVC 是作为另一个 MVC 的 V 部分而存在的，也就是说，父 MVC 中的 C 控制的是子 MVC 中的 C，而不是直接去干预子 MVC 中的 V。当然，多个 MVC 是可以共享同一个 M 的，毕竟 Notification 是广播性质的，可以有多个 Observer。


[1]: https://github.com/ReactiveCocoa/ReactiveCocoa