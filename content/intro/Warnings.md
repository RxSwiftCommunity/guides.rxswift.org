+++
date = "2015-10-23T16:12:42+02:00"
title = "Advertencias"
categories = "introducción"
tags = ["observables", "guia", "operators", "API", "documentación"]
+++

### <a name="unused-disposable"></a>Desechable no usado (unused-disposable)

Lo siguiente es valido para toda la familia de funciones `subscribe*`, `bind*` y `drive*` que devuelven un `Disposable` (desechable).

La advertencia probablemente se presente en un contexto similar a este:

```Swift
let xs: Observable<E> ....

xs
    .filter { ... }
    .map { ... }
    .switchLatest()
    .subscribe(onNext: {
        ...
    }, 
    onError: {
        ...
    })  
```

La función `subscribe` devuelve una suscripción `Disposable` (desechable) que se puede utilizar para cancelar el cálculo y liberar recursos.

Forma preferida de poner fin al flujo de llamadas, es mediante el uso de `.addDisposableTo(disposeBag)` o de manera equivalente

```Swift
let xs: Observable<E> ....
let disposeBag = DisposeBag()

xs
    .filter { ... }
    .map { ... }
    .switchLatest()
    .subscribe(onNext: {
        ...
    }, 
    onError: {
        ...
    })
    .addDisposableTo(disposeBag) // <--- observe `addDisposableTo`
```

Cuando se desasigna `disposeBag`, todas las suscripciones serán automáticamente desechadas.

En el caso de `xs` termine de una manera predecible con `Completed` o `Error`, aunque no se controlando la suscripción `Disposable` no se pierde ningún recurso, pero sigue siendo la manera preferido porque de esta forma los cálculos del elemento se terminarán en un momento predecible.

Esto también hará que su código robusto y resistente al paso del tiempo ya que los recursos serán desechados correctamente aunque `xs` cambie de implementación.

Otra forma de asegurarse de que las suscripciones y los recursos están vinculados con la vida útil de un objeto es el uso de operador `takeUntil`.

```Swift
let xs: Observable<E> ....
let someObject: NSObject  ...

_ = xs
    .filter { ... }
    .map { ... }
    .switchLatest()
    .takeUntil(someObject.rx_dellocated) // <-- observe el operador `takeUntil` 
    .subscribe(onNext: {
      ...
    }, onError: {
      ...
    })
```

Si lo que desea es ignorar la suscripción `Disposable`, así es como se silencia la advertencia del compilador.

```Swift
let xs: Observable<E> ....
let disposeBag = DisposeBag()

_ = xs // <-- observe el guión bajo
    .filter { ... }
    .map { ... }
    .switchLatest()
    .subscribe(onNext: {
      ...
    }, onError: {
      ...
    })
```

### <a name="unused-observable"></a>Unused observable sequence (unused-observable)

Probablemente aparecerá una advertencia en un contexto similar a este:

```Swift
let xs: Observable<E> ....

xs
    .filter { ... }
    .map { ... }
```

Este código define secuencia observable que se filtra y se asigna a la secuencia `xs` pero ignora el resultado.

Dado que este código sólo define una secuencia observable y luego lo ignora, en realidad no hacen nada y es bastante inútil.

Su intención era, probablemente, almacenar la definición de secuencia observable y usarla más tarde ...

```Swift
let xs: Observable<E> ....

let ys = xs // <--- nombra la definición como `ys`
    .filter { ... }
    .map { ... }
```

... o iniciar un cálculo basandose en esa definición 

```Swift
let xs: Observable<E> ....
let disposeBag = DisposeBag()

xs
    .filter { ... }
    .map { ... }
    .subscribeNext { nextElement in       // <-- note the `subscribe*` method
      ... probably print or something
    }
    .addDisposableTo(disposeBag)
```
