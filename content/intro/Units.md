+++
date = "2015-10-23T16:12:42+02:00"
title = "Unidades (Units)"
categories = "introducción"
tags = ["observables", "guide", "API", "documentation"]
+++

Este documento tratará de describir lo que son las unidades (Units), y porque son un concepto útil, cómo usarlos y cómo crearlos.

* [¿Por qué?](#¿Por-qué?)
* [¿Cómo funcionan?](##¿Cómo-funcionan?)
* [¿Por qué se han llamado Unidades?](##¿Por-qué-se-han-llamado-Unidades?)
* [Unidades RxCocoa](##Unidades-RxCocoa)
* [Unidad Driver](##Unidad-Driver)
    * [¿Por qué fue nombrado Driver?](###¿Por-qué-fue-nombrado-Driver?)
    * [Practical usage example](#practical-usage-example)

##Por qué

Swift tiene un sistema de tipo de gran alcance que se puede utilizar para mejorar la exactitud y estabilidad de las aplicaciones y hacer uso de Rx una experiencia más intuitiva y sencilla.

**Las unidades son hasta el momento específico sólo para el proyecto (RxCocoa)[https://github.com/ReactiveX/RxSwift/tree/master/RxCocoa ], pero los mismos principios podrían aplicarse fácilmente en otras implementaciones Rx si es necesario. No se necesita mágia de la API privada.**

**Las unidades son totalmente opcionales, puede utilizar secuencias observables brutas por todas partes en su programa y todas las API RxCocoa trabajar con secuencias observables.**

Las unidades también ayudan a comunicar y asegurar la secuencia de propiedades observables a través de límites de interfaz.

Estas son algunas de las propiedades que son importantes al escribir aplicaciones Cocoa/UIKit.

* No puede fallar
* Se observar en planificador principal
* Se suscribe en planificador principal
* Se comparten los efectos secundarios

##¿Cómo funcionan?

En su núcleo es sólo una estructura con una referencia a la secuencia observable.

Suede pensar en ellas como una especie de patrón de constructor para secuencias observables. Cuando se construye la secuencia, llamando `.asObservable()` se transformará en una unidad en una secuencia observable corriente.

##¿Por qué se han llamado Unidades?

Analogías ayudan a razonar sobre conceptos desconocidos, aquí están algunas ideas de cómo las unidades de la física y RxCocoa (rx units) son similares.

Analogías:

| Unidades físicas                      | Rx units                                                                |
|---------------------------------------|-------------------------------------------------------------------------|
| numero (un valor)                     | secuencia observable (secuencia de valores)                             |
| unidad dimensional (m, s, m/s, N ...) | Estructura Swift (Driver, ControlProperty, ControlEvent, Variable, ...) |

Unidad física es un par compuesto de un número y una unidad de dimensión correspondiente. <br/>
Unidad de Rx es un par de una secuencia observable y una estructura correspondiente que describe la secuencia de propiedades observables.

Los números son el pegamento de la composición básica cuando se trabaja con unidades físicas:. Los números generalmente reales o complejos.<br/>
Secuencias observables son el pegamento de composición básico cuando se trabaja con unidades de rx.

Unidades físicas y [análisis dimensional](https://en.wikipedia.org/wiki/Dimensional_analysis#Checking_equations_that_involve_dimensions) puede aliviar cierta clase de errores durante cálculos complejos. <br/>
La comprobación de tipos le las unidades rx puede aliviar cierta clase de errores lógicos al escribir programas reactivos.

Los números tienen los operadores: `+`, `-`, `*`, `/`<br/>.
Las secuencias observables también tienen operadores: `map`, `filter`, `flatMap` ...

Unidades Física definen las operaciones mediante el uso de las operaciones numéricas correspondientes. Por ejemplo.

La operación `/` en unidades físicas se define utilizando la operación `/` en los números.

11 m / 0,5 s = ...
* Primero se convierten la unidades a **números** y se **aplica** el operador `/` `11 / 0,5 = 22`
* Luego calcular la unidad (m / s)
* Combinar el resultado = 22 m / s

Unidades Rx definen las operaciones mediante el uso de secuencias correspondientes operaciones observables (así es como operadores trabajan internamente). Por ejemplo.

El operacion `map` sobre `Driver` se define utilizando la operación `map` en su secuencia observable.

```swift
let d: Driver<Int> = Drive.just(11)
driver.map { $0 / 0.5 } = ...
```

* Primero se convierte driver a **secuencia observable** y **aplica** el **operator** `map`
```swift
let mapped = driver.asObservable().map { $0 / 0.5 } // este `map` se define en la secuencia observable
```

* luego combina eso para conseguir el valor de la unidad
```swift
let result = Driver(mapped)
```

Hay un conjunto de unidades básicas de la física [(`m`, `kg`, `s`, `A`, `K`, `cd`, `mol`)](https://en.wikipedia.org/wiki/SI_base_unit) que son ortogonales.<br/>
Hay una serie de interesantes propiedades básicas de las secuencias observables en `RxCocoa` que son ortogonales.

    * No puede fallar
    * Se observar en planificador principal
    * Se suscribe en planificador principal
    * Se comparten los efectos secundarios

Las unidades derivadas de la física a veces tienen nombres especiales.<br/>
Por ejemplo:
```
N (Newton) = kg * m / s / s
C (Coulomb) = A * s
T (Tesla) = kg / A / s / s
```

Las unidades derivadas de Rx también tienen nombres especiales.<br/>
Por ejemplo:
```
Driver = (no puede fallar) * (observar en planificador principal) * (comparten los efectos secundarios)
ControlProperty = (comparten los efectos secundarios) * (suscribe en planificador principal)
Variable = (no puede fallar) * (comparten los efectos secundarios)
```

La conversión entre diferentes unidades de física se realiza con la ayuda de los operadores definidos en los números `*`, `/`. <br/>
La conversión entre diferentes unidades de RX en hecho con una ayuda de los operadores de secuencia observables.

Por ejemplo:

```
no puede fallar = catchError
observar el planificador principal = observeOn(MainScheduler.instance)
suscribir el planificador principal = subscribeOn(MainScheduler.instance)
compartiendo efectos secundarios = share* (uno de los operadores `share`)
```


## Unidades RxCocoa

### Driver

* no puede fallar
* se observar en planificador principal
* comparten los efectos secundarios (`shareReplayLatestWhileConnected`)

### ControlProperty / ControlEvent

* no puede fallar
* se suscribe en planificador principal
* se observar en planificador principal
* comparten los efectos secundarios

### Variable

* no puede fallar
* comparten los efectos secundarios

## Unidad Driver

Esta es la unidad más elaborada. Su intención es proporcionar una forma intuitiva para escribir código reactiva en la capa de interfaz de usuario.

### ¿Por qué fue nombrado Driver?

Su caso uso previsto es modelar secuencias que impulsan su aplicación.

Por ejemplo:
* Conduce la IU desde un modelo CoreData
* Conduce la IU enlazando los valores de otro elemento del IU (bindings)
...


Al igual que los conductores normales del sistema operativo, en caso de que uno de esos errores de secuencia fuera su aplicación será dejar de responder a la entrada del usuario.

También es muy importante que esos elementos se observan en el hilo principal, porque los elementos de interfaz de usuario y la lógica de aplicación son por lo general no seguro para subprocesos.

También, la unidad `Driver` construye una secuencia que comparte los efectos secundarios del observable.

Por ejemplo:

### Practical usage example

Este es un ejemplo típico de principiante.

```swift
let results = query.rx_text
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
    }

results
    .map { "\($0.count)" }
    .bindTo(resultCount.rx_text)
    .addDisposableTo(disposeBag)

results
    .bindTo(resultsTableView.rx_itemsWithCellIdentifier("Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .addDisposableTo(disposeBag)
```


El comportamiento previsto de este código fue:
* Estrangular la entrada de usuario
* Servidor de contacto y buscar una lista de resultados usuario (una vez por consulta)
* Luego unen los resultados a dos elementos de la interfaz de usuario, los resultados vista de tabla y una etiqueta que muestra el número de resultados

¿Cuáles son los problemas con este código:
* En caso de que la secuencia observable `fetchAutoCompleteItems` no se lleve a cabo (la conexión falla o hay un error de análisis), este error podría desatar todo y la interfaz de usuario no respondería más a las nuevas consultas.
* En caso de que la secuencia observable `fetchAutoCompleteItems` devuelve resultados en hilo de segundo planoo, los resultados estarían obligados a elementos de interfaz de usuario desde un subproceso de fondo y que podrían causar accidentes no deterministas.
* Los resultados están vinculados a dos elementos de interfaz de usuario, lo que significa que por cada usuario consulta dos peticiones HTTP se hizo, una para cada elemento de la interfaz de usuario, que no pretende comportamiento.


Una versión más apropiado del código se vería así:

```swift
let results = query.rx_text
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
            .observeOn(MainScheduler.instance) // los resultados se devuelven en MainScheduler
            .catchErrorJustReturn([])          // en el peor de los casos, los errores se manejan
    }
    .shareReplay(1)                            // Las peticiones HTTP son compartidas y los resultados se reproducen
                                               //  a todos los elementos de IU

results
    .map { "\($0.count)" }
    .bindTo(resultCount.rx_text)
    .addDisposableTo(disposeBag)

results
    .bindTo(resultTableView.rx_itemsWithCellIdentifier("Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .addDisposableTo(disposeBag)
```

Asegurarse de que todos estos requisitos son manejados adecuadamente en grandes sistemas puede ser un reto, pero hay una manera más sencilla de utilizar el compilador y unidades de demostrar que se cumplan estos requisitos.

El siguiente código es casi igual:

```swift
let results = query.rx_text.asDriver()        // Esto convierte una secuencia normal en una secuencia `Driver`.
    .throttle(0.3, scheduler: MainScheduler.instance)
    .flatMapLatest { query in
        fetchAutoCompleteItems(query)
            .asDriver(onErrorJustReturn: [])  // El constructor sólo necesita la información de que devolver en caso de error.
    }

results
    .map { "\($0.count)" }
    .drive(resultCount.rx_text)               // Esa disponible el método `drive` en lugar de `bindTo`,
    .addDisposableTo(disposeBag)              // eso significa que el compilador ha comprobado que se satisfacen 
                                              // todas las propiedades.
results
    .drive(resultTableView.rx_itemsWithCellIdentifier("Cell")) { (_, result, cell) in
        cell.textLabel?.text = "\(result)"
    }
    .addDisposableTo(disposeBag)
```

Entonces, ¿qué está pasando aquí?

Este primer método `asDriver` convierte la unidad `ControlProperty` a una unidad `Driver`.

```swift
query.rx_text.asDriver()
```

Tenga en cuenta que no había nada especial que había que hacer. `Driver` tiene todas las propiedades de la unidad de `ControlProperty` además de algunas más. La secuencia observable subyacente se acaba de envolver como una unidad `Driver`, y eso es todo.



El segundo cambio es:

```swift
  .asDriver(onErrorJustReturn: [])
```

Cualquier secuencia observable se puede convertir en una unidad `Driver`, sólo tiene que satisfacer a 3 propiedades:
* no puede fallar
* se debe observar en el planificador principal (`MainScheduler.instance`)
* debe compartir los efectos secundarios (`shareReplayLatestWhileConnected`)

Entonces, ¿cómo asegurarse de que esas propiedades están satisfechas? Sólo tiene que utilizar operadores normales Rx.`asDriver(onErrorJustReturn: [])` es equivalente al siguiente código.

```
let safeSequence = xs
  .observeOn(MainScheduler.instance)       // observar eventos en planificador principal
  .catchErrorJustReturn(onErrorJustReturn) // no puede fallar
  .shareReplayLatestWhileConnected         // compartir los efectos secundarios
return Driver(raw: safeSequence)           // lo envuelve
```

La pieza final es usar `drive` lugar de `bindTo`.
`drive` se define sólo en la unidad `Driver`. Esto significa que si usted ve `drive` algún lugar en el código, secuencia observable que nunca puede error y observa elementos en hilo principal está siendo obligados a elemento de la interfaz de usuario. Que es exactamente lo que se quiere.
En teoría, alguien podría definir el método `drive` para trabajar en `ObservableType` o alguna otra interfaz, por lo que la creación de una definición temporal con `let results: Driver<[Results]> = ...` antes de la unión a elementos de la IU sería necesario para la prueba completa, pero lo dejaremos para lector a decidir si esto es un escenario realista.

