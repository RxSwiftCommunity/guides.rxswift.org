# 快速开始

这个项目尽量与[ReactiveX.io](http://reactivex.io/)保持一致。一个常规的关于`RxSwift`跨平台的文档跟指南。



1. [Observables 又称 Sequences](#Observables 又称 Sequences)
2. [销毁](#销毁)
3. [隐式观察者保证（Implicit `Observable` guarantees）](# 隐式观察者保证（Implicit `Observable` guarantees）)
4. [创建你自己的观察者`Observable`](# 创建你自己的观察者`Observable`)
5. []()

# Observables 又称 Sequences

## 基础

有关于序列（`Generator`）跟观察者模式（`Observable<Element>`）的[等价性]()是理解`Rx`的最重要的部分之一。

观察者模式是必要的，因为你需要对异步行为进行建模，这等价于在`Observable`上实现高阶序列操作符

序列是一个简单的常见的概念因为**很容易去想象**。

人类拥有强大的视觉皮层，所以可以很容易的去想象一些东西。

通过模仿其他Rx系列里面在序列上的高级操作，你可以简化很多已有的工作。

如果你不利用Rx对你的异步系统建模，这可能意味着你的代码模充斥着需要被模拟的状态机跟临时状态，而正确的方式是抽象他们。

序列可能是程序员认识到的一种思想。

这是一组数字序列

``` 
--1--2--3--4--5--6--| // 正常的终止
```

这是一组字符的序列

``` 
--a--b--a--a--a---d---X // 以一个错误终止
```

有些序列是有限的，有些是无限的，比如一组按钮的点击事件

``` 
---tap-tap-------tap--->
```

​	

这些图解被叫做[marble diagrams](http://rxmarbles.com/)



如果制定一个语法规定，他应该是这样的

**Next\*(Error|Completed)**

他可以被描述成

- 序列有0个或者多个元素
- 一次接受一个`Error`或者`Complete`事件，序列不会生成其他的元素

Rx中的序列都提供了一种`push interface`（又称回调）

``` Swift
	enum Event<Element>  {
    case Next(Element)      // 序列中下一个元素
    case Error(ErrorType)   // 序列以失败结束
    case Completed          // 序列正常结束
}

class Observable<Element> {
    func subscribe(observer: Observer<Element>) -> Disposable
}

protocol ObserverType {
    func on(event: Event<Element>)
}
```

当序列发送了`Complete`或者`Error`事件时，序列中所有计算过的资源都会被释放。

对于被返回的序列可以调用`dispose`来释放资源跟终止序列的运行。

如果序列在某个时间暂停了并且没有调用`dispose`也没有调用`addDisposableTo(disposeBag)`,这并不会引起永久性资源泄露，但是这些资源直到序列以某种方式（执行完所有事件或者发生了错误）终止才会被使用。

如果序列没有终止，资源将会被永久占用除非`dispose`被手动调用，或者自动调用内部的`disposeBag`，`scopedDispose`,`takeUntil`或者其他的方式。

**使用`disposeBag`，`scopedDispose`,`takeUntil`操作都是确保资源被释放的有效地方式。尽管序列在最后也会被终止，我们也推荐使用它们。**

也许你会好奇为什么`ErrorType`不是泛型，你可以在[这里](/intro/DesignRationale/#why-error-type-isnt-generic)找到解释。

# 销毁

 这里有一种方式来销毁一个观察序列。当你完成了一个序列并且想要释放所有的资源以分配给即将到来的事件，只需要在一个订阅上调用`dispose`即可。

这里有一个关于`interval`的例子

``` Swift
let subscription = interval(0.3, scheduler)
    .subscribe { (e: Event<Int64>) in
        print(e)
    }

NSThread.sleepForTimeInterval(2)

subscription.dispose()

```

输出：

``` 
0
1
2
3
4
5
```

直的注意的是你通常是不想手动调用`dispose`的，并且这是一个有教育意义的例子。手动调用`dispose`的写法是不好的，并且还有一个更好的方式去销毁一个订阅。你可以使用`DisposeBag`,`ScopedDisposable`或者`takeUntil`中的任意一个操作或者其他的途径来销毁。

那么当这段代码执行完`dispose`还会打印其他东西吗？答案是，他会取决于：

- 如果`scheduler`是**串行调度**的（`mainScheduler`是串行调度），并且`dispose`在这**同一个串行调度程序**上调用，那么答案是不会打印。
  
- 其他情况下会打印
  
  ​

你可以在[这里](http://guides.rxswift.org/intro/Schedulers/)看到更多关于调度的资料。

你可以简单地理解为两个进程同时运行

- 一个是生产元素
- 一个是销毁订阅

当你思考它的时候，这个`还会打印其他东西`的问题甚至还没确定是否在不同的调度程序上。

一些例子正好来证明（`observeOn`的解释在[这里](intro/Schedulers/)）

假设你有一些代码

``` Swift
let subscription = interval(0.3, scheduler)
            .observeOn(MainScheduler.sharedInstance)
            .subscribe { (e: Event<Int64>) in
                print(e)
            }

// ....

subscription.dispose() // 在主线程被调用

```

能确定的是，在`dispose`调用之后不会有任何输出。

再比如说这个例子

``` Swift
let subscription = interval(0.3, scheduler)
            .observeOn(serialScheduler)
            .subscribe { (e: Event<Int64>) in
                print(e)
            }

// ...

subscription.dispose() // 在相同的 `serialScheduler` 被执行
```

能确定的是，在`dispose`调用之后不会有任何输出。

## Dispose Bags

Dispose bags are used to return ARC like behavior to RX.

当`DisposeBag`被释放的时候，他将会对每一个被添加的订阅调用`dispose`

它没有一个`dispose`函数并且不允许直接调用dispose，如果需要立即释放只需要创建一个新的bag

``` Swift
self.disposeBag = DisposeBag()
```

这将会清楚所有的资源

如果仍然需要手动的清理资源，用`CompositeDisposable`，他有你想要的效果，但是一旦`dispose`函数被调用，他将会立刻销毁最近被添加的订阅。

## Scoped Dispose

如果想要在离开作用域之后立刻被销毁，可以用`scopedDispose()`

``` Swift
let autoDispose = sequence
    .subscribe {
        print($0)
    }
    .scopedDispose()
```

当执行完离开作用域后，订阅将立刻被销毁。

## Take until

其他的自动销毁订阅的方式还有`takeUntil`操作

``` Swift
sequence
    .takeUntil(self.rx_deallocated)
    .subscribe {
        print($0)
    }
```

# 隐式观察者保证（Implicit `Observable` guarantees）

There is also a couple of additional guarantees that all sequence producers (`Observable`s) must honor.

不论在哪个线程生产元素，如果他们生成一个元素并且发送给了观察者`observer.on(.Next(nextElement))`,他们就只能等到`observer.on`函数执行完才能发送下一个元素。

被观察者也不能发送`.Complete`或者`.Error` 当`.Next`没有结束的时候。

总之，看下面的例子

``` Swift
someObservable
  .subscribe { (e: Event<Element>) in
      print("Event processing started")
      // processing
      print("Event processing ended")
  }
```

只会打印

``` 
Event processing started
Event processing ended
Event processing started
Event processing ended
Event processing started
Event processing ended
```

永远不会打印：

``` 
Event processing started
Event processing started
Event processing ended
Event processing ended
```

# 创建你自己的观察者`Observable`

这里有一个重要的事情去理解被观察者

**当一个被观察者被创建的时候，它不会因为简单地被创建了就去执行任何工作。**

事实是被观察者`Observable`可以通过很多方式来创建元素。有些会产生负作用(cause side effects),有些会利用现有的进程比如利用鼠标事件，等等。

但是如果你调用了一个返回`Observable`的函数，没有任何序列会执行它，也没有任何副作用。`Observable`只是一个序列被产生的定义，并且也被当作序列中的元素。当`subscribe`函数被调用的时候序列才会执行。

举一个例子：

``` Swift
func searchWikipedia(searchTerm: String) -> Observable<Results> {}
```

``` Swift
let searchForMe = searchWikipedia("me")

// no requests are performed, no work is being done, no URL requests were fired

let cancel = searchForMe
  // sequence generation starts now, URL requests are fired
  .subscribeNext { results in
      print(results)
  }

```

有很多种创建`Observable`序列的方式，也许最简单的就是使用`create`函数。

让我们创建一个只返回一个元素的序列，这个函数叫做`just`

这是一个具体的实现：

``` 
func myJust<E>(element: E) -> Observable<E> {
    return create { observer in
        observer.on(.Next(element))
        observer.on(.Completed)
        return NopDisposable.instance
    }
}

myJust(0)
    .subscribeNext { n in
      print(n)
    }
```

这将会输出：

``` 
0
```

不错，这就是`create`函数

它只是一个能让你使用Swift的闭包简单的实现`subscribe`函数的一种简便方法。比如`subscribe`函数带了一个参数`observer`并且返回可被回收的对象（disposable）。

以这种方式实现的序列实际上是同步的，它在`subscribe`函数调用之前产生元素并且终止然后返回可被回收的订阅。返回的可被回收的订阅是无关紧要的因为进程产生元素的过程不能被中断。

当产生一个同步序列，通常返回的可被回收订阅是一个`NopDisposable`的单例实例。

我们现在从一个数组创建一个被观察者返回一系列元素。

这是一个实现：

``` 
func myFrom<E>(sequence: [E]) -> Observable<E> {
    return create { observer in
        for element in sequence {
            observer.on(.Next(element))
        }

        observer.on(.Completed)
        return NopDisposable.instance
    }
}

let stringCounter = myFrom(["first", "second"])

print("Started ----")

// first time
stringCounter
    .subscribeNext { n in
        print(n)
    }

print("----")

// again
stringCounter
    .subscribeNext { n in
        print(n)
    }

print("Ended ----")
```

将会输出

``` 
Started ----
first
second
----
first
second
Ended ----
```

# Creating an Observable that performs work








## Operators 操作符

There are numerous operators implemented in RxSwift. The complete list can be found [here](API).
RxSwift中定义并实现了多种多样的操作符。完整的操作符列表可以在这里查看[here](API).

Marble diagrams for all operators can be found on [ReactiveX.io](http://reactivex.io/)
为了消除一些歧义我们使用Marble diagrams去区分所有的操作符。这个图表可以在这里查看到[ReactiveX.io](http://reactivex.io/)

Almost all operators are demonstrated in [Playgrounds](https://github.com/ReactiveX/RxSwift/tree/master/Rx.playground).
绝大多数的操作符展示和事例可以在这个playground查看到。[Playgrounds](https://github.com/ReactiveX/RxSwift/tree/master/Rx.playground).

To use playgrounds please open `Rx.xcworkspace`, build `RxSwift-OSX` scheme and then open playgrounds in `Rx.xcworkspace` tree view.
如果您想使用playgrounds, 首先您需要请先打开 `Rx.xcworkspace`, 然后build `RxSwift-OSX` scheme，之后在项目树中打开playgrounds. 之后你就看可以正常使用playgrounds了.

In case you need an operator, and don't know how to find it there a [decision tree of operators]() http://reactivex.io/documentation/operators.html#tree).
如果当你需要一个操作符但是不知道如何找到它, 您可以查阅以下资料[decision tree of operators]() http://reactivex.io/documentation/operators.html#tree).

[Supported RxSwift operators](API#rxswift-supported-operators) are also grouped by function they perform, so that can also help.
[Supported RxSwift operators](API#rxswift-supported-operators) 这些操作符同样也以功能进行了分组, 所以它也对你的开发有所帮助.

### Custom operators 自定义操作符

There are two ways how you can create custom operators.
这里我们有两种方式让你创建自己想要的操作符.

#### Easy way 第一种: 简单的方式

All of the internal code uses highly optimized versions of operators, so they aren't the best tutorial material. That's why it's highly encouraged to use standard operators.
所有的内部代码都是使用高度优化的操作符进行编写的，所以这些代码不是最好的参考材料. 那也是为什么我们鼓励使用普通的操作符。

Fortunately there is an easier way to create operators. Creating new operators is actually all about creating observables, and previous chapter already describes how to do that.
庆幸的是这里有一种简单的方式去创建自定义操作符. 事实上, 创建一个新的操作符完全在创建各式各样的observables。至于如何创建observables, 我们已经在前一章节提及到.

Lets see how an unoptimized map operator can be implemented.
让我们先来看看一个未被优化的map操作符是怎杨实现的.

```swift
func myMap<E, R>(transform: E -> R)(source: Observable<E>) -> Observable<R> {
    return create { observer in

        let subscription = source.subscribe { e in
                switch e {
                case .Next(let value):
                    let result = transform(value)
                    observer.on(.Next(result))
                case .Error(let error):
                    observer.on(.Error(error))
                case .Completed:
                    observer.on(.Completed)
                }
            }

        return subscription
    }
}
```

So now you can use your own map:
所以现在你能够使用你自己的map操作符了:

```swift
let subscription = myInterval(0.1)
    .myMap { e in
        return "This is simply \(e)"
    }
    .subscribeNext { n in
        print(n)
    }
```

and this will print
之后它将打印出如下结果

```
Subscribed
This is simply 0
This is simply 1
This is simply 2
This is simply 3
This is simply 4
This is simply 5
This is simply 6
This is simply 7
This is simply 8
...
```

#### Harder, more performant way 第二种: 创建更加困难, 但是性能更好的方法

You can perform the same optimizations like we have made and create more performant operators. That usually isn't necessary, but it of course can be done.
你能够使用跟我们一样的方式进行优化进而创建更多有效率的操作符. 通常情况下, 虽然这个不是必要的, 但是它能达到我们想要的优化目的. 

Disclaimer: when taking this approach you are also taking a lot more responsibility when creating operators. You will need to make sure that sequence grammar is correct and be responsible of disposing subscriptions.
免责声明: 当你采用这种方式进行创建自定义操作符，你需要为你所创建的操作符负更多的责任.并且 你还将需要保证你的序列语法是正确的并且为撤销订阅负责.


There are plenty of examples in RxSwift project how to do this. I would suggest talking a look at `map` or `filter` first.
在RxSwift项目中有很多这方面的例子去创建自定义操作符. 在这里, 我推荐你们可以先查看和学习 `map` 或者 `filter`操作符.

Creating your own custom operators is tricky because you have to manually handle all of the chaos of error handling, asynchronous execution and disposal, but it's not rocket science either.
由于你需要人工的进行处理自定义操作符所带来的各式各样的麻烦, 比如如何处理错误, 异步执行和销毁问题.所以创建自定义操作符是很困难的, 虽说困难但是这也不会像火箭工程学那么困难.

Every operator in Rx is just a factory for an observable. Returned observable usually contains information about source `Observable` and parameters that are needed to transform it.
Rx中的每一个操作符仅仅是一个observable工厂. 被返回的observable通常包涵了可以改变其自己的source `Observable` 和参数.(这句有出路)


In RxSwift code, almost all optimized `Observable`s have a common parent called `Producer`. Returned observable serves as a proxy between subscribers and source observable. It usually performs these things:
在Rxswift的代码中, 绝大多数的已优化的 `Observable`s 都有一个公共的父类 `Producer`. 被返回的observable作为订阅者(subscribers)和source observable之间的代理, 它通常需要完成如下几件事情:


* on new subscription creates a sink that performs transformations
* 在新的subscription上创建一个sink来进行transformations

* registers that sink as observer to source observable
* 把这个sink作为一个观察者(observer)注册到source observable

* on received events proxies transformed events to original observer
* 在收到事件的代理上传输事件给原始的observer
