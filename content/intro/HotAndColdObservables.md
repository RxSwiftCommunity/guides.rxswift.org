+++
date = "2015-10-23T16:12:42+02:00"
title = "Hot and Cold Observables"
categories = "introduction"
tags = ["observables", "guide", "documentation"]
+++

Hot and Cold Observables
========================

从我的角度来看, 我建议把这个Hot and Cold Observables看作是sequences的一个特性, 不是分离开来的不同类型. 因为他们都可以完美的抽象为`Observable` sequence.

下面是Hot and Cold Observables在ReactiveX.io中的定义

> 何时一个Observable开始发送它的items序列? 这个将要依赖于the Observable. 一个“hot” Observable一旦被创建,它可能立即发送items, 因此之后订阅这个Observable的任何observer可能是从sequence的流程中的某个时刻开始监视这个sequence(即可能无法监视整个sequence). 另一方面, 一个“cold” Observable将会等待, 直到有一个observer订阅了它, 它才开始发送这些items, 因此这将保证observer能够观察到从头开始的整个sequence的过程.

| Hot Observables                                                                                         | Cold observables                                                              |
|---------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| ... 是sequences                                                                                       | ... 是sequences                                                             |
| 使用资源("produce heat") 不管是否有observer订阅它                         | 不使用资源 (don't produce heat) 直到有observer订阅它.           |
| Variables(变量) / properties(属性) / constants(常量), tap coordinates(点击的坐标), mouse coordinates(鼠标的坐标), UI control values(UI控制的参数), current time(当前时间) | Async operations(异步操作), HTTP Connections(HTTP连接), TCP connections(TCP连接), streams(流)                  |
| 通常包涵 ~ N个元素(elements)                                                                    | 通常包含 ~ 1个元素(element)                                                  |
| 无论是否有observer订阅, 都会产生sequence elements                           | 仅当有observer订阅, 才会产出Sequence.        |
| 通常情况下的Sequence计算资源在所有观察者(observers)之间共享 are usually shared between all of the subscribed observers.              | 通常情况下的Sequence计算资源被分配给每个订阅的observer. |
| 通常是 stateful                                                                                        | 通常是 stateless                                                             |
