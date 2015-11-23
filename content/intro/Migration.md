+++
date = "2015-10-23T16:12:42+02:00"
title = "Migración de la versión RxSwift 1.9 a 2.0"
categories = "introducción"
tags = ["observables", "observers", "guide", "API", "documentation"]
+++

La migración debería ser bastante sencillo. Los cambios son principalmente cosméticos, por lo que todas las funciones están todavía allí.

* Remplaza cualquier `>- ` por `.`
* Remplaza cualquier "variable" por "shareReplay(1)"
* Remplaza cualquier "catch" por "catchErrorJustReturn"
* Remplaza cualquier "returnElement" por "just"
* Como nos hemos movido de `>-` a `.`, funciones libres son ahora los métodos, por lo que `.switchLatest()`, `.distinctUntilChanged()`, ... en vez de `>- switchLatest`, `>- distinctUntilChanged`
* Como nos hemos movido de funciones libres a extensiones por lo que ahora es `[a, b, c].concat()`, `.merge()`, ... donde era `concat([a, b, c])`, `merge(sequences)`
* Ahora es `subscribe { n in ... }.addDisposableTo(disposeBag)` en vez de `>- disposeBag.addDisposable`
* El método `next` de Variable ahora es el setter `value`
* Si desea utilizar `tableViews`/`collectionViews`, este es el caso de uso básico ahora

```swift
viewModel.rows
            .bindTo(resultsTableView.rx_itemsWithCellIdentifier("WikipediaSearchCell")) { (_, viewModel, cell: WikipediaSearchCell) in
                cell.viewModel = viewModel
            }
            .addDisposableTo(disposeBag)
```

Si usted tiene más dudas de cómo escribir algún concepto en la versión 2.0 de RxSwift, echale un ojo a [la App de ejemplo](https://github.com/ReactiveX/RxSwift/tree/master/RxExample) o a los [playgrounds](https://github.com/ReactiveX/RxSwift/tree/master/Rx.playground).
