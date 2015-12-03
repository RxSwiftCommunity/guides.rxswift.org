+++
date = "2015-10-23T16:12:42+02:00"
title = "Units"
categories = "introduction"
tags = ["observables", "guide", "API", "documentation"]
+++

Units
=====

这份文档将阐述什么是Units, 为什么Units是一个有用的概念, 还有如何创建和使用他们.

* [Why](#为什么)
* [Design Rationale](#设计的逻辑依据)
* ...

# 为什么

使用Units的目的是使用Swift的编译器的静态类型检查去确保你的代码的行为表现与你所设计的行为表现保持一致.

RxCocoa项目已经包涵了多种units, 但是最详尽且用心当属`Driver`, 所以这个`Driver`unit将被作为解释units背后设计想法的例子.

`Driver` 描述着驱动应用程序中特定的一些组成部分的sequences, 所以这个unit以`Driver`命名. 这些sequences不仅仅将会驱动UI bindings和驱动可以保证你的应用的交互性的UI event pumps, 还会驱动应用服务(application services)等等...

使用`Driver` unit的目的是保证the underlying observable sequence有着如下的特性.

* 不会失败, 所有的错误都将完整的处理
* 元素(elements)在主线程中传递
* 序列的计算资源(sequence computation resources)是共享的

TBD...
