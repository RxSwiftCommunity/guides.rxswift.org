+++
date = "2015-10-23T16:12:42+02:00"
title = "Migration from RxSwift 1.9 to RxSwift 2.0 version"
categories = "introduction"
tags = ["observables", "observers", "guide", "API", "documentation"]
+++

#从RxSwift 1.9版本迁移至RxSwift 2.0版本

从1.9版本迁移至2.0版本是非常直观方便的. 版本之间的改动是很有限的, 所以所有的功能特性在2.0版本中依然存在.

* 查找项目中所有的`>- ` 并替换成 `.`
* 查找项目中所有的 "variable" 并替换成 "shareReplay(1)"
* 查找项目中所有的 "catch" 并替换成 "catchErrorJustReturn"
* 查找项目中所有的 "returnElement" 并替换成 "just"
* 自从我们把`>-` 变成 `.`之后, 现在函数(free functions)变成的方法(methods), 所以这里举出以下列子 `.switchLatest()`, `.distinctUntilChanged()`, ... 这两个方法替换了 `>- switchLatest`, `>- distinctUntilChanged`
* 我们已经把函数(free functions)变成了拓展(extensions), 所以现在这些`[a, b, c].concat()`, `.merge()`, ... 替换了原来的旧的 `concat([a, b, c])`, `merge(sequences)`
* 现在 `subscribe { n in ... }.addDisposableTo(disposeBag)` 替换了原来的 `>- disposeBag.addDisposable`
* `Variable`的方法(method) `next` 现在改为 `value` setter
* 假如说你想使用`tableViews`/`collectionViews`, 现在基本的使用方法是如下的代码:

```swift
viewModel.rows
            .bindTo(resultsTableView.rx_itemsWithCellIdentifier("WikipediaSearchCell")) { (_, viewModel, cell: WikipediaSearchCell) in
                cell.viewModel = viewModel
            }
            .addDisposableTo(disposeBag)
```

假如你还有一些关于怎么编写RxSwift 2.0 代码的疑问, 请查阅以下相关资料 [Example app](https://github.com/ReactiveX/RxSwift/tree/master/RxExample) or [playgrounds](https://github.com/ReactiveX/RxSwift/tree/master/Rx.playground).
