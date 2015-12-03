+++
date = "2015-10-23T16:12:42+02:00"
title = "Math Behind Rx"
categories = "introduction"
tags = ["guide", "documentation"]
+++

## 观察者(Observer)和迭代器(Iterator) / 枚举器(Enumerator) / 生成器(Generator) / 序列(Sequences)之间的二元性

observer模式和generator模式之间存在二元性. 那是能够使得异步回调的世界变成sequence同步transformations世界的东西.

简而言之, 枚举(enumerator)和观察者(observer)都是描述了sequences. 对于enumerator, enumerator中定义了sequence是很明显的

对于observer这里也有一个没有包涵数学概念并且很简明的解释: 假设你正在监视鼠标的运动. 每一次你收到的鼠标移动都是一个鼠标移动序列中的一员.

简而言之, 这里有两个能被使用的序列中的基本元素.

* Push interface - Observer (observed elements over time make a sequence)
* Pull interface - Iterator / Enumerator / Generator

想要学习这方面的内容, 以下的一些视频将会对您有所帮助

你也能够在这些视频中看到一些更加正式但不失风趣的解释

[Expert to Expert: Brian Beckman and Erik Meijer - Inside the .NET Reactive Framework (Rx) (video)](https://www.youtube.com/watch?v=looJcaeboBY)
