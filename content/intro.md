# 快速开始

这个项目尽量与[ReactiveX.io](http://reactivex.io/)保持一致。一个常规的关于`RxSwift`跨平台的文档跟指南。

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

# 创建一个可以执行的`Observable`



现在有趣的来了，让我么你创建一个用于前一个例子的`interval`操作

这种方式与调度队列（dispatch queue schedulers）实现起来是等价的。

``` Swift
func myInterval(interval: NSTimeInterval) -> Observable<Int> {
    return create { observer in
        print("Subscribed")
        let queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
        let timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue)

        var next = 0

        dispatch_source_set_timer(timer, 0, UInt64(interval * Double(NSEC_PER_SEC)), 0)
        let cancel = AnonymousDisposable {
            print("Disposed")
            dispatch_source_cancel(timer)
        }
        dispatch_source_set_event_handler(timer, {
            if cancel.disposed {
                return
            }
            observer.on(.Next(next++))
        })
        dispatch_resume(timer)

        return cancel
    }
}
```



``` Swift
let counter = myInterval(0.1)

print("Started ----")

let subscription = counter
    .subscribeNext { n in
       print(n)
    }

NSThread.sleepForTimeInterval(0.5)

subscription.dispose()

print("Ended ----")
```



输出结果：

``` 
Started ----
Subscribed
0
1
2
3
4
Disposed
Ended ----
```



如果你这样写：

``` Swift
let counter = myInterval(0.1)

print("Started ----")

let subscription1 = counter
    .subscribeNext { n in
       print("First \(n)")
    }
let subscription2 = counter
    .subscribeNext { n in
       print("Second \(n)")
    }

NSThread.sleepForTimeInterval(0.5)

subscription1.dispose()

NSThread.sleepForTimeInterval(0.5)

subscription2.dispose()

print("Ended ----")
```

结果则是这样的：

``` 
Started ----
Subscribed
Subscribed
First 0
Second 0
First 1
Second 1
First 2
Second 2
First 3
Second 3
First 4
Second 4
Disposed
Second 5
Second 6
Second 7
Second 8
Second 9
Disposed
Ended ----
```

**每一个订阅了得订阅者都会有各自的用来产生元素的序列。函数默认是无状态的。无状态的函数要比有状态的多得多。**

# 共享订阅和`shareReplay`函数

如果你想要多个观察者从一个订阅共享事件（元素）应该怎么做呢？

有两件事需要明确：

- 如何处理在新的订阅者订阅之前已经接收到的元素。（重新接受最新的一个，重新接受左右，还是重新接受最新的n个）
- 何时释放被共享的订阅（引用计数（refCount），手动释放（manual），或者是其他算法）

通常的选择是结合`replay(1).refCount()`或者`shareReplay()`

``` Swift
let counter = myInterval(0.1)
    .shareReplay(1)

print("Started ----")

let subscription1 = counter
    .subscribeNext { n in
       print("First \(n)")
    }
let subscription2 = counter
    .subscribeNext { n in
       print("Second \(n)")
    }

NSThread.sleepForTimeInterval(0.5)

subscription1.dispose()

NSThread.sleepForTimeInterval(0.5)

subscription2.dispose()

print("Ended ----")
```

输出：

``` 
Started ----
Subscribed
First 0
Second 0
First 1
Second 1
First 2
Second 2
First 3
Second 3
First 4
Second 4
First 5
Second 5
Second 6
Second 7
Second 8
Second 9
Disposed
Ended ----
```

注意这里只打印了一次`Subscribed`跟`Disposed`事件。

跟URL被观察者（URL observables）的行为是一样的。

这是HTTP请求被封装成Rx。这跟`interval`函数非常相似。

``` Swift
extension NSURLSession {
    public func rx_response(request: NSURLRequest) -> Observable<(NSData!, NSURLResponse!)> {
        return create { observer in
            let task = self.dataTaskWithRequest(request) { (data, response, error) in
                if data == nil || response == nil {
                    observer.on(.Error(error ?? UnknownError))
                }
                else {
                    observer.on(.Next(data, response))
                    observer.on(.Completed)
                }
            }

            task.resume()

            return AnonymousDisposable {
                task.cancel()
            }
        }
    }
}
```

## 函数（Operators）

RxSwift中定义并实现了多种多样的函数。完整的函数列表可以在这里查看[here](API).

为了消除一些歧义我们使用Marble diagrams去区分所有的操作符。这个图表可以在这里查看到[ReactiveX.io](http://reactivex.io/)

绝大多数的函数展示和事例可以在这个playground查看到。[Playgrounds](https://github.com/ReactiveX/RxSwift/tree/master/Rx.playground).

如果您想使用playgrounds, 首先您需要请先打开 `Rx.xcworkspace`, 然后build `RxSwift-OSX` scheme，之后在项目树中打开playgrounds. 之后你就看可以正常使用playgrounds了.

如果当你需要一个函数但是不知道如何找到它, 您可以查阅以下资料[decision tree of operators](http://reactivex.io/documentation/operators.html#tree).

[Supported RxSwift operators](API#rxswift-supported-operators) 这些操作符同样也以功能进行了分组, 所以它也对你的开发有所帮助.

### Custom operators 自定义函数

这里我们有两种方式让你创建自己想要的函数.

#### 第一种: 简单的方式

所有的内部代码都是使用高度优化的函数进行编写的，所以这些代码不是最好的参考材料. 那也是为什么我们鼓励使用普通的函数。

庆幸的是这里有一种简单的方式去创建自定义函数. 事实上, 创建一个新的函数完全在创建各式各样的observables。至于如何创建observables, 我们已经在前一章节提及到.

让我们先来看看一个未被优化的map函数是怎么实现的.

``` swift
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

所以现在你能够使用你自己的map操作符了:

``` swift
let subscription = myInterval(0.1)
    .myMap { e in
        return "This is simply \(e)"
    }
    .subscribeNext { n in
        print(n)
    }
```

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

#### 第二种: 创建更加困难, 但是性能更好的方法

你能够使用跟我们一样的方式进行优化进而创建更多有效率的函数. 通常情况下, 虽然这个不是必要的, 但是它能达到我们想要的优化目的. 

免责声明: 当你采用这种方式进行创建自定义函数，你需要为你所创建的函数负更多的责任.并且 你还将需要保证你的序列语法是正确的并且为撤销订阅负责.

在RxSwift项目中有很多这方面的例子去创建自定义函数. 在这里, 我推荐你们可以先查看和学习 `map` 或者 `filterh`函数.

由于你需要人工的进行处理自定义函数所带来的各式各样的麻烦, 比如如何处理错误, 异步执行和销毁问题.所以创建自定义操作符是很困难的, 虽说困难但是这也不会像火箭工程学那么困难.

Every operator in Rx is just a factory for an observable. Returned observable usually contains information about source `Observable` and parameters that are needed to transform it.

Rx中的每一个函数仅仅是一个observable工厂. 被返回的observable通常包涵了可以改变其自己的source `Observable` 和参数.(这句有出路)

在Rxswift的代码中, 绝大多数的已优化的 `Observable`s 都有一个公共的父类 `Producer`. 被返回的observable作为订阅者(subscribers)和source observable之间的代理, 它通常需要完成如下几件事情:

* on new subscription creates a sink that performs transformations
* 在新的subscription上创建一个sink来进行transformations
* registers that sink as observer to source observable
* 把这个sink作为一个观察者(observer)注册到source observable
* on received events proxies transformed events to original observer
* 在收到事件的代理上传输事件给原始的observer

## 特殊情况（Life happens）

So what if it’s just too hard to solve some cases with custom operators? You can exit the Rx monad, perform actions in imperative world, and then tunnel results to Rx again using `Subject`s.

如果在一些特殊情况下使用普通函数解决问题很麻烦。你可以不使用Rx单子（Rx monad），（ perform actions in imperative world），然后通过使用`Subject`s把结果返回Rx。

这个不会经常用到。并且这是一种很差的代码，但是你可以使用它。

``` Swift
 let magicBeings: Observable<MagicBeing> = summonFromMiddleEarth()

  magicBeings
    .subscribeNext { being in     // exit the Rx monad  
        self.doSomeStateMagic(being)
    }
    .addDisposableTo(disposeBag)

  //
  //  Mess
  //
  let kitten = globalParty(   // calculate something in messy world
    being,
    UIApplication.delegate.dataSomething.attendees
  )
  kittens.on(.Next(kitten))   // send result back to rx
  //
  // Another mess
  //

  let kittens = Variable(firstKitten) // again back in Rx monad

  kittens
    .map { kitten in
      return kitten.purr()
    }
    // ....
```

每次你这样做的时候，其他人可能会在某处写上如下代码：

``` Swift
 kittens
    .subscribeNext { kitten in
      // so something with kitten
    }
    .addDisposableTo(disposeBag)
```

所以尽量别这样做。

# Playgrounds

如果你不确定其中的一些函数式如何工作的，[playgrounds](https://github.com/ReactiveX/RxSwift/tree/master/Rx.playground)包含了了几乎所有的函数以及用来举例的Demo。

**使用playground之前先打开Rx.xcworkspace, 编译 RxSwift-OSX scheme 并且在 Rx.xcworkspace 目录中playgroupnds。**

如果想在playgrounds看到所有Demo的结果，请打开`Assistant Editor`。你可以通过`View > Assistant Editor > Show Assistant Editor`来打开`Assistant Editor`  

# 错误处理

有两种错误处理机制

## 被观察者中的异步错误处理机制（Asynchronous error handling mechanism in observables）

错误处理是很简单的。如果一个序列被一个错误终止，所有的依赖序列都会被错误终止。这是一种简单的逻辑。

你可以通过`catch`函数从被观察者的错误恢复。有很多种重载可以使你恢复到合适的地方。

不仅如此，还有一个`retry`函数让你重新执行(retry)出错的序列。

# 调试编译错误

当你写RxSwift/RxCocoa代码的时候，你可能严重的依赖编译器对``Observable`的类型推倒。这也正是一个Swift很棒的原因，但是伴随而来的也有不好的东西。

``` Swift
images = word
    .filter { $0.rangeOfString("important") != nil }
    .flatMap { word in
        return self.api.loadFlickrFeed("karate")
            .catchError { error in
                return just(JSON(1))
            }
      }
```

如果编译器报告了一个错误，我会首先会假设返回类型有问题。

``` Swift
images = word
    .filter { s -> Bool in s.rangeOfString("important") != nil }
    .flatMap { word -> Observable<JSON> in
        return self.api.loadFlickrFeed("karate")
            .catchError { error -> Observable<JSON> in
                return just(JSON(1))
            }
      }
```

如果这不管用，你可以继续添加更多额外的类型直到解决这个错误。

``` Swift
images = word
    .filter { (s: String) -> Bool in s.rangeOfString("important") != nil }
    .flatMap { (word: String) -> Observable<JSON> in
        return self.api.loadFlickrFeed("karate")
            .catchError { (error: NSError) -> Observable<JSON> in
                return just(JSON(1))
            }
      }
```

**我首先会假设返回类型跟闭包参数**

通常你解决错误之后，你可以移除这些额外的类型来再一次简化你的代码。

# 调试

单独使用调试工具是有效的，但是你还可以使用`debug`函数。`debug`函数会以标准输出打印出所有的事件并且你可以对事件添加标标签。

`debug`用起来像一个探针一样。这里有一个例子。

``` Swift
let subscription = myInterval(0.1)
    .debug("my probe")
    .map { e in
        return "This is simply \(e)"
    }
    .subscribeNext { n in
        print(n)
    }

NSThread.sleepForTimeInterval(0.5)

subscription.dispose()
```

输出：

``` 
[my probe] subscribed
Subscribed
[my probe] -> Event Next(Box(0))
This is simply 0
[my probe] -> Event Next(Box(1))
This is simply 1
[my probe] -> Event Next(Box(2))
This is simply 2
[my probe] -> Event Next(Box(3))
This is simply 3
[my probe] -> Event Next(Box(4))
This is simply 4
[my probe] dispose
Disposed
```

你还可以使用`subscribe`来代替`subscribeNext`

``` Swift
NSURLSession.sharedSession().rx_JSON(request)
   .map { json in
       return parse()
   }
   .subscribe { n in      // this subscribes on all events including error and completed
       print(n)
   }
```

# 调试内存泄露

在调试模式下Rx会用全局变量`resourceCount`来记录所有分配的资源。

**在push一个view controller到导航控制器之后打印`Rx.resourceCount`，使用它， 然后pop出来是一个调试内存泄露的好办法。**

一个明智的做法是在view controller的`deinit`方法中执行`print`方法。

代码大概是这样的：

``` Swift
class ViewController: UIViewController {
#if TRACE_RESOURCES
    private let startResourceCount = RxSwift.resourceCount
#endif

    override func viewDidLoad() {
      super.viewDidLoad()
#if TRACE_RESOURCES
        print("Number of start resources = \(resourceCount)")
#endif
    }

    deinit {
#if TRACE_RESOURCES
        print("View controller disposed with \(resourceCount) resources")

        var numberOfResourcesThatShouldRemain = startResourceCount
        let time = dispatch_time(DISPATCH_TIME_NOW, Int64(0.1 * Double(NSEC_PER_SEC)))
        dispatch_after(time, dispatch_get_main_queue(), { () -> Void in
            print("Resource count after dealloc \(RxSwift.resourceCount), difference \(RxSwift.resourceCount - numberOfResourcesThatShouldRemain)")
        })
#endif
    }
}
```

之所以使用一个短暂的延时是因为有时候需要等待实体（scheduld entities）花一点时间去释放他们的内存。

# 变量（Variables）

variables 代表了一些被观察者的状态。`Variable`如果值就不会存在因为构造器（initializer）需要一个初始值。

Variable 封装了一个`Subject`。特别说明的是他是一个`BehaviorSubject`。不像`BehaviorSubject`，他仅仅暴漏了`value`接口，所以variable永远不会终止跟失败。

他也是一被订阅就会广播他当前的值。

``` Swift
let variable = Variable(0)

print("Before first subscription ---")

variable
    .subscribeNext { n in
        print("First \(n)")
    }

print("Before send 1")

variable.value = 1

print("Before second subscription ---")

variable
    .subscribeNext { n in
        print("Second \(n)")
    }

variable.value = 2

print("End ---")
```

输出：

``` 
Before first subscription ---
First 0
Before send 1
First 1
Before second subscription ---
Second 1
First 2
Second 2
End ---
```

# KVO

