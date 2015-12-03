+++
date = "2015-10-23T16:12:42+02:00"
title = "Design Rationale"
categories = "introduction"
tags = ["error type", "Design", "Event", "guide", "API", "documentation"]
+++

设计的逻辑依据
================

## 为什么error type不是泛型

```Swift
enum Event<Element>  {
    case Next(Element)      // next element of a sequence
    case Error(ErrorType)   // sequence failed with error
    case Completed          // sequence terminated successfully
}
```

首相让我们先探讨下当`ErrorType`是泛型时候的优缺点

假如说你有一个错误类型(Error Type)的泛型, 你将在两个observables之间造成一个额外的impedance mismatch.

让我们来举个列子, 假如你你有以下的两个Observable:

`Observable<String, E1>` 和 `Observable<String, E2>`

当你没办法指出明确的error type的情况下, 你将没有办法使用和处理这两个Observable.

因为这个错误类型可能是`E1`, `E2`或者是一个全新的错误类型`E3`? 你需要一些新的函数操作(operators)去处理这类的impedance mismatch.

这种设计将有害于composition properties, 并且Rx并不关心sequence是如何出现错误, 它仅仅只是将这些错误信息推送到the observable chain.

而且在一些情况下这种设计也会造成一些额外的问题. 比如在一些情况下operators可能会因为内部错误而失败, 并且在这种情况下你没办法构建错误信息,也就也没办法反馈和获得错误报告.

但是这一切都还好, 让我们先忽略这种问题. 假设我们能够根据之前的问题建模一种不会抛出error的sequences. 从这种情况来看, 这种sequence可能对处理之前的那种问题有效?

是的, 这种sequence是有处理这个问题的潜能, 但是首先让我们先思考下为什么我们需要一个不会抛出error的sequences.

一个明确的应用是UI层里的permanent streams. 但是当你仔细思考这个例子的时候, 你会发现单单使用编译器去证明sequence没有错误抛出是完全不够的, 你还需要证明其他的properties也没有错误. 就像`MainScheduler`中的被观察的elements一样.

你所需要的是一中普遍的方法去验证sequences (`Observables`)的特点. 并且你能够对特性感到兴趣. 例如:

* sequence在有限的时间内终结(服务器端)
* sequence包涵仅有一个元素element (假如你正在运行一些计算)
* sequence不error out, 永远不终止并且elements在mian scheduler中传递 (UI)
* sequence不error out, 永远不终止并且elements在mian scheduler中传递再者已经被refcounted sharing (UI)
* sequence不error out, 永远不终止并且elements在特定的background scheduler中传递 (audio engine)

你所真正需要的是一个普通的,能够为了observable sequences强制执行系统特性的编译器, 并且还需要一些处理想要的properties的一些不变的函数(operators).

从我的角度来看, 以下的例子就是一个很好的类比:

```
1, 3.14, e, 2.79, 1 + 1i      <->    Observable<E>
1m/s, 1T, 5kg, 1.3 pounds     <->    Errorless observable, UI observable, Finite observable ...
```

在Swift中, 有很多方式通过composition或inheritance of observables来实现.

使用unit system的好处还有就是你能够保证UI代码能在同一个scheduler中执行因此对于所有的transformations使用无锁的operators

自从Rx已经没有对单一的sequence operations的locks, 并且那些所遗留下来的locks都存在于有状态的组件(statefull components)里(又称UI), 因此实际上从Rx代码中移除所有的剩下的locks并且创建一个编译器强制执行无锁的(lockless)Rx代码.

所以对我而已, 在保留Rx组合语意(Compositional Semantics)的前提下,使用不能够在其他方面实现更加简洁的typed Errors是没有益处的. 并且另外的一些方法也有其巨大的好处.
