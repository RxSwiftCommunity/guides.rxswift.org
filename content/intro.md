+++
date = "2015-10-23T16:12:42+02:00"
title = "Introducción"
categories = "Introducción"
tags = ["observables", "guia", "introducción", "documentación"]
+++

Este proyecto trata de ser coherente con [ReactiveX.io](http://reactivex.io/). Documentación general multi-plataforma y tutoriales que deberian ser válidos para el caso de RxSwift.

# Observables tambien conocidos como secuencias

## Basics
Es [equivalence](MathBehindRx) al patrón observados pattern (`Observable<Elemento>`) y secuencias (`Generator`s) es una de las cosas más importantes de comprender sobre Rx.

Se necesita el patrón Observer porque se quiere modelar el comportamiento asíncrono y 
que la equivalencia permita la ejecución de operaciones de secuencias de alto nivel como 
operadores para los `Observable`s.

Las secuencias son un concepto simple y familiar que es **fácil de visualizar**.

Las personas son criaturas con un gran córtex visual. Cuando se puede visualizar algo fácil, es mucho más fácil de razonar sobre ello.

De esa manera usted puede mejorar gran cantidad de la carga cognitiva al tratar de simular las máquinas de estado de evento dentro de cada operador de Rx a las operaciones de alto nivel sobre las secuencias.

Si no utiliza Rx pero modela sistemas asincrónicos, probablemente signifique que su código esté lleno de estas máquinas de estado y estados transitorios necesitara simularlos en lugar de abstraerlos.

Listas/secuencias son probablemente uno de los primeros conceptos que matemáticos/programadores aprenden.

Tenemos una secuencia de números


```
--1--2--3--4--5--6--| // que termina normalmente
```

Tenemos otra con caracteres

```
--a--b--a--a--a---d---X // pero esta termina con error
```

Algunas secuencias son finitas, y otras son infinitas, como la secuencia de toques de un botón

```
---tap-tap-------tap--->
```

Estos diagramas se llaman diagramas mármol.

[http://rxmarbles.com/](http://rxmarbles.com/)

Si tuviéramos que especificar la gramática de la secuencia como una expresión regular sería algo como esto

**Next* (Error | Completed)**

Esto describe lo siguiente:

* **las secuencias pueden tener 0 o más elementos**
* **Una vez un evento `Error` o `Completed` es recivido, la secuencia ya no podrá producir ningún otro elemento**

Secuencias en Rx se describen mediante una interfaz de empuje (o devolución de llamada).

```swift
enum Event<Element>  {
    case Next(Element)      // el elemento siguiente de la secuencia
    case Error(ErrorType)   // la secuencia falla y produce un error
    case Completed          // la secuencia termina con éxito
}

class Observable<Element> {
    func subscribe(observer: Observer<Element>) -> Disposable
}

protocol ObserverType {
    func on(event: Event<Element>)
}
```

**Cuando una secuencia envia un evento `Complete` o `Error` se liberan todos los recursos internos que computan los elementos de la secuencia.**


**Para cancelar la producción de elementos de secuencia y liberar los recursos inmediatemente, se llama al metodo `dispose` de la suscripción devuelta.**

Si una secuencia termina en tiempo finito, no llamar a `dispose` o no usar `addDisposableTo(disposeBag)` no causará ningúna pérdida de recursos permanentes, pero esos recursos serán utilizados hasta que la secuencia complete de alguna manera (dejando de producir elementos o sucediendo un error).

Si una secuencia no termina de alguna manera, los recursos se asignarán de forma permanente a menos que se llame a `dispose` manualmente, automáticamente dentro de un `disposeBag`, `scopedDispose`, `takeUntil` o de alguna otra manera.

**Usar dispose bags, scoped dispose o el operador `takeUntil` son maneras robustas de asegurarse de que los recursos estan limpios y recomendamos su uso en la producción a pesar de que la secuencia terminará en un tiempo finito**

En caso de que usted es curioso por qué `ErrorType` no es genérico, se puede encontrar una explicación [aquí](DesignRationale#why-error-type-isnt-generic).

## Desechando

Hay una forma adicional de que una secuencia observada pueda terminar. Cuando se haya terminado con una secuencia y desee liberar todos los recursos que se asignaron para calcular elementos siguientes, llamando al metodo `dispose` de una suscripción hara esto por usted.

He aquí un ejemplo con el operador `interval`.

```swift
let subscription = interval(0.3, scheduler)
    .subscribe { (e: Event<Int64>) in
        print(e)
    }

NSThread.sleepForTimeInterval(2)

subscription.dispose()

```

Esto imprimirá:

```
0
1
2
3
4
5
```

Una cosa a destacar aquí es que por lo general no se deseará llamar a `dispose` manualmente y esto es únicamente un ejemplo educativo. Llamar a disponer manualmente suele ser código que huele mal, y hay mejores maneras de eliminar las suscripciones. Usted puede usar `DisposeBag`,` ScopedDisposable`, `operador takeUntil` o algún otro mecanismo.

Entonces, ¿puede el código imprimir algo después de que la llamada a `dispose` se ejecute? La respuesta es, depende.

* Si el `scheduler` (planificador) es **serial scheduler** (`MainScheduler` es un serial scheduler) y `dispose` es llamado **en el mismo serial scheduler**, entonces la respuesta es **no**.

* de lo contrario **sí**.

Se puede encontrar más información sobre los schedulers [aquí](Schedulers).

Usted simplemente tiene dos procesos que suceden en paralelo.

* uno esta produciendo elementos
* el otro esta desechando suscripciones

Cuando se piensa en ello, la pregunta `puede algo ser impreso despues` ni siquiera tiene sentido en el caso de que esos procesos se encuentran en diferentes planificadores.

Algunos ejemplos más para estar seguro (`observeOn` se explica [aquí](Schedulers)).

En caso de que tenga algo como:

```swift
let subscription = interval(0.3, scheduler)
            .observeOn(MainScheduler.sharedInstance)
            .subscribe { (e: Event<Int64>) in
                print(e)
            }

// ....

subscription.dispose() // llamado desde la tarea principal

```

**Despues de que se devuelva la llamada a `dispose`, nada será impreso. Esto esta garantizado.**

Tambien en este caso:

```swift
let subscription = interval(0.3, scheduler)
            .observeOn(serialScheduler)
            .subscribe { (e: Event<Int64>) in
                print(e)
            }

// ...

subscription.dispose() // ejecutandolo en el mismo `serialScheduler`

```

**Despues de que se devuelva la llamada a `dispose`, nada será impreso. Esto esta garantizado.**

### Dispose Bags (Bolsas de desechado)

Las Dispose bags se utilizan para dar a RX un comportamiento como ARC.

Cuando `DisposeBag` se desasigna, se llamará a `dispose` en cada uno de los desechables añadidos.

No tiene un método `dispose` y no permite llamar desechar explícitamente a propósito. Si se necesita una limpieza inmediata basta con crear una nueva bolsa.

```swift
  self.disposeBag = DisposeBag()
```

Eso debería borrar las referencias al viejo y por lo tanto causar la eliminación de los recursos.

Si aun necesita la eliminación manual explícitamente, use `CompositeDisposable`. **Tiene el comportamiento deseado, pero una vez que se llama al metodo `dispose`, se desechara de inmediato cualquier desechable que se le añada.**

### Scoped Dispose

En caso de que la eliminación se quiera inmediatamente después de abandonar el alcance de la ejecución, hay `scopedDispose()`.

```swift
let autoDispose = sequence
    .subscribe {
        print($0)
    }
    .scopedDispose()
```

Esto desechará la suscripción cuando la ejecución deje el ámbito actual.

### Take until

Otra manera adicional para desechar la suscripción automáticamente en dealloc es usar el operador `takeUntil`.

```swift
sequence
    .takeUntil(self.rx_deallocated)
    .subscribe {
        print($0)
    }
```

<a name="implicit-observable-guarantees" ></a>
## Garantías implicitas en los `Observable`

También hay un par de garantías adicionales que todos los productores de secuencia (`Observable`s) deben cumplir.

No importa en qué hilo que produzcan los elementos, pero si generan un elemento y lo envían al observador `observer.on(.Next(nextElement))`, no pueden enviar el elemento siguiente hasta que el metodo `observer.on` termine la ejecución.

Los productores tampoco pueden enviar que estan concluidos con `.Completed` o `.Error` en caso que el evento `.Next` no haya terminado.

En resumen, considere este ejemplo:

```swift
someObservable
  .subscribe { (e: Event<Element>) in
      print("Event processing started")
      // processing
      print("Event processing ended")
  }
```

esto siempre imprimirá:

```
Event processing started
Event processing ended
Event processing started
Event processing ended
Event processing started
Event processing ended
```

y nunca puede imprimir:

```
Event processing started
Event processing started
Event processing ended
Event processing ended
```

## Creación de su propio `Observable` (o secuencia observable)

Hay una cosa fundamental que entender sobre los observables.

**Cuando se crea un observable, no se realiza ningún trabajo simplemente porque haya sido creado.**

Es cierto que `Observable` puede generar elementos de muchas maneras. Algunos de ellos causan efectos secundarios y algunos de ellos aprovechan los procesos en ejecución existentes, como aprovechar los eventos de ratón, etc.

**Pero si unicamente llama a un método que devuelve un `Observable`, no se realiza ninguna generación de la secuencia, y no hay efectos secundarios. `Observable` es sólo una definición de cómo se genera la secuencia y los parámetros que se utilizan para la generación de elemento. La generación de la secuencia comienza cuando se llama al método `subscribe`.**

Por ejemplo. Digamos que usted tiene un método con un prototipo similar a este:

```swift
func searchWikipedia(searchTerm: String) -> Observable<Results> {}
```

```swift
let searchForMe = searchWikipedia("me")

// no se realizan peticiones, no se está trabajando, no se dispararon peticiones de URL

let cancel = searchForMe
  // la generación de la secuencia comienza ahora, se disparan las solicitudes de URL
  .subscribeNext { results in
      print(results)
  }

```

Hay un montón de maneras de cómo usted puede crear su propia secuencia `Observable`. Probablemente la forma más fácil es usar la función `create`.

Vamos a crear una función que crea una secuencia que devuelve un unico elemento en el momento de la suscripción. Esa función se llama 'just'.

*Esta es la implementación real*

```swift
func myJust<E>(element: E) -> Observable<E> {
    return create { observer in
        observer.on(.Next(element))
        obsever.on(.Completed)
        return NopDisposable.instance
    }
}

myJust(0)
    .subscribeNext { n in
      print(n)
    }
```

esto va a imprimir:


```
0
```

No está mal. Entonces, ¿Qué es la función `create`?

Es sólo un método de conveniencia que le permite implementar fácilmente el método `subscribe` usando la función lambda Swift. Al igual que el método `subscribe` toma un argumento, `observer`, y devuelve un desechable.

La secuencia implementada de esta manera es en realidad síncrona. Generará elementos y terminará antes de la llamada a `subscribe` devuelva el desechable que representa la suscripción. Debido a que en realidad no importa el desechable que devuelva, el proceso de generación de elementos no puede ser interrumpido.

Al generar secuencias sincrónicas, normalmente el desachable a devolver es una instancia singleton de `NopDisposable`.

Vamos ahora a crear un observable que devuelve los elementos de una matriz.

*Esta es la implementación actual*

```swift
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

// la primera vez
stringCounter
    .subscribeNext { n in
        print(n)
    }

print("----")

// otra vez
stringCounter
    .subscribeNext { n in
        print(n)
    }

print("Ended ----")
```

Esto imprimirá:

```
Started ----
first
second
----
first
second
Ended ----
```

## Creación de un `Observable` que realiza trabajos

Ok, ahora algo más interesante. Vamos a crear ese operador `interval` que se utilizó en los ejemplos anteriores.

*Esto es equivalente de la implementación actual para los dispatch queue schedulers*


```swift
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

```swift
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

Esto imprimirá:

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

¿Qué pasa si se escribiese?

```swift
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

esto podría imprimir:

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

**Cada suscriptor una vez se suscribe suele generar su propia secuencia separada de elementos. Los operadores no tienen estado de forma predeterminada. Hay muchos más operadores sin estado que con estado**

## Compartiendo la suscripción y el operador `shareReplay`

Pero si lo que se desea es que varios observadores compartan los eventos (elementos) de una misma suscripción?

Hay dos cosas que necesitan ser definidas.

* Cómo manejar los anteriores elementos que fueron recibidos antes de que el nuevo suscriptor estubiese interesado en la observación de ellos (repetir solo el último, repetir todos, repetir los últimos n).
* ¿Cómo decidir cuándo disparar esa suscripción compartida (refCount, manual o algún otro algoritmo)

La elección habitual es una combinación de `replay(1).refCount()` que es lo mismo que `shareReplay()`.

```swift
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

esto imprimirá:

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

Observe cómo ahora sólo hay un evento `Subscribed` y un `Disposed`.

El comportamiento de los observables de URL es equivalente.

Es así como las peticiones HTTP se envuelven en Rx. Es más o menos el mismo patrón que con el operador `interval`.

```swift
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

## Los operadores

Hay numerosos operadores implantados en RxSwift. La lista completa se puede encontrar [aquí](API).

Diagramas Marble para todos los operadores se pueden encontrar en [ReactiveX.io](http://reactivex.io/)

Casi todos los operadores se demuestran en [Playgrounds](https://github.com/ReactiveX/RxSwift/tree/master/Rx.playground).

Para utilizar los Playgrounds por favor abra `Rx.xcworkspace`, contruya el esquema `RxSwift-OSX` y entonces seleccione Rx.playground en la vista de árbol del workspace.

En caso de que necesite un operador, y no sepa cómo encontrarlo en [árbol de decisión de los operadores](http://reactivex.io/documentation/operators.html#tree) (en inglés).

[Los operadores soportados por RxSwift](API#rxswift-supported-operators) también se agrupan por la función que realizan, por lo que también puede ayudar.

### Operadores a medida

Hay dos maneras de cómo se puede crear operadores personalizados.

#### El camino fácil

Todo el código interno utiliza versiones de los operadores altamente optimizados, por lo que no son el mejor material como tutorial. Es por eso que es altamente recomendable que use los operadores estándar.

Afortunadamente hay una manera más fácil de crear operadores. Creación de nuevos operadores en realidad es todo acerca de la creación observables, y el capítulo anterior ya se describe cómo hacerlo.

Vamos a ver cómo un operador map` no optimizado puede ser implementado.

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

Así que ahora usted puede utilizar su propio `map`:

```swift
let subscription = myInterval(0.1)
    .myMap { e in
        return "This is simply \(e)"
    }
    .subscribeNext { n in
        print(n)
    }
```

y esto imprimirá

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

#### La manera complicada, pero más eficientemente

Puede realizar las mismas optimizaciones como lo hemos hecho y crear operadores más eficientes. Que por lo general no es necesario, pero por supuesto se puede hacer.

Exención de responsabilidad: Al elegir este enfoque esta a su vez tomando mucha más responsabilidad al crear los Operadores. Deberá asegurarse la gramatica de la sequencia es correcta y es responsable de desechar las suscrupciones.

Hay un montón de ejemplos de cómo hacer esto en el proyecto RxSwift. Yo sugeriría echarle un vistazo a `map` o `filter` primero.

La creación de sus propios operadores personalizados es complicado porque hay que manejar manualmente todo el caos de la gestión de errores, la ejecución asíncrona y el desechado, pero no es ciencia de cohetes tampoco.

Cada operador Rx es sólo una fábrica de un observable. Devuelven observables que por lo general contienen información sobre la fuente `Observable` y los parámetros que se necesitan para transformarla.

En el código RxSwift, casi todos los `Observable`s optimizados tienen un padre común que se llama `Producer`. El Observable devuelto sirve como un proxy entre los suscriptores y el observable fuente. Por lo general, realiza estas cosas:

* on new subscription creates a sink that performs transformations
* registers that sink as observer to source observable
* on received events proxies transformed events to original observer

### La vida pasa

¿Y qué si es demasiado difícil de resolver algunos casos con operadores personalizados? Puede salir de la mónada Rx, realizar acciones en el mundo imperativo, a continuación, los resultados de los túneles a Rx nuevamente utilizando `Subject`s.

Esto no es algo que debe ser practicado a menudo, y es código que huele mal, pero se puede hacer.

```swift
let magicBeings: Observable<MagicBeing> = summonFromMiddleEarth()

magicBeings
    .subscribeNext { being in // aqui estamos saliendo de la mónada Rx  
        self.doSomeStateMagic(being)
    }
    .addDisposableTo(disposeBag)

//
//  Desorden
//
let kitten = globalParty(   // calcula algo en el mundo desordenado
        being,
        UIApplication.delegate.dataSomething.attendees
    )
kittens.on(.Next(kitten))   // envia el resultado de nuevo a rx

//
// Otro desorden
//
let kittens = Variable(firstKitten) // otra vez de vuelta a la mónada Rx

kittens
    .map { kitten in
        return kitten.purr()
    }
// ....
```

Cada vez que hace esto, alguien probablemente escribió este código en alguna parte

```swift
kittens
    .subscribeNext { kitten in
      // así algo con kitten
    }
    .addDisposableTo(disposeBag)
```

así que por favor trate de no hacer esto.

## Playgrounds

Si no está seguro de cómo funcionan exactamente algunos de los operadores, [playgrounds](https://github.com/ReactiveX/RxSwift/tree/master/Rx.playground) contiene casi todos los operadores ya preparados con pequeños ejemplos que ilustran su comportamiento.

**Para utilizar los Playgrounds por favor abra `Rx.xcworkspace`, contruya el esquema `RxSwift-OSX` y entonces seleccione Rx.playground en la vista de árbol del workspace.**

**Para ver los resultados de los ejemplos en los Playgrounds, por favor, abra el 'Assistant Editor`. Puede abrir `Assistant Editor` haciendo clic en 'View > Assistant Editor > Show Assistant Editor`**

## Manejo de errores

Existen dos mecanismos de error.

### Mecanismo asincrono de manejo de errores en los observables

El manejo de errores es bastante sencillo. Si una secuencia termina con error, a continuación, todas las secuencias dependientes dará por terminada con error. Es la lógica de cortocircuito habitual.

Puede recuperarse de fracaso de observables mediante el uso del operador `catch`. Hay varias sobrecargas que le permiten especificar la recuperación en gran detalle.

También esta el operador `retry` que permite reintentos en caso de secuencia termine con error.

### Manejo de errores síncrono

Desafortunadamente Swift no tiene un concepto de excepciones o algún tipo de construcción en mónada de error por lo que este proyecto introduce la enumeración `RxResult`.

Es el portado a Swift del tipo [`Try`](http://www.scala-lang.org/api/2.10.2/index.html#scala.util.Try) de Scala. También es similar a la mónada [`Either`](https://hackage.haskell.org/package/category-extras-0.52.0/docs/Control-Monad-Either.html) de Haskell.

**Esto será reemplazado en Swift 2.0 con try/throws**

```swift
public enum RxResult<ResultType> {
    case Success(ResultType)
    case Error(ErrorType)
}
```

Para habilitar la escritura de código más legible, se introducen unos pocos operadores `Result`

```swift
result1
    .flatMap { okValue in    // bloque de manejo de éxito
        // ejecutado cuando hay éxito
        return ?
    }
    .recoverWith { error in  // bloque de manejo de error
        // ejecutado cuando hay error
        return ?
    }
```

## Depuración de errores de compilación

Al escribir código RxSwift/RxCocoa elegante, es probable que la fuerte dependencia del compilador para inferir los tipos de los `Observable`s. Esta es una de las razones por las Swift es impresionante, pero también puede ser frustrante a veces.

```swift
images = word
    .filter { $0.rangeOfString("important") != nil }
    .flatMap { word in
        return self.api.loadFlickrFeed("karate")
            .catchError { error in
                return just(JSON(1))
            }
      }
```

Si el compilador informa que hay un error en algún lugar de esta expresión, sugeriría primer anotar los tipos de retorno.

```swift
images = word
    .filter { s -> Bool in s.rangeOfString("important") != nil }
    .flatMap { word -> Observable<JSON> in
        return self.api.loadFlickrFeed("karate")
            .catchError { error -> Observable<JSON> in
                return just(JSON(1))
            }
      }
```

Si eso no funciona, puede continuar agregando más anotaciones de tipo hasta que se localice el error.

```swift
images = word
    .filter { (s: String) -> Bool in s.rangeOfString("important") != nil }
    .flatMap { (word: String) -> Observable<JSON> in
        return self.api.loadFlickrFeed("karate")
            .catchError { (error: NSError) -> Observable<JSON> in
                return just(JSON(1))
            }
      }
```

**Se sugiere que primero se anoten los tipos de retorno y los argumentos de los cierres.**

Por lo general, después de que haya solucionado el error, puede eliminar las anotaciones de tipo para limpiar su código de nuevo.

## Depuración

Usar el depurador es útil, pero también se puede usar el operador `debug`. El operador `debug` imprimirá todos los eventos en la salida estándar y se puede añadir también etiquetar esos eventos.

`debug` actúa como una sonda. Aquí hay un ejemplo de su uso:

```swift
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

imprimirá

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

También se puede usar `subscribe` en lugar de `subscribeNext`

```swift
NSURLSession.sharedSession().rx_JSON(request)
   .map { json in
       return parse()
   }
   .subscribe { n in      // esto se suscribe en todos los eventos, incluyendo error y completado
       print(n)
   }
```

## Depurando fugas de memoria

En el modo de depuración Rx hace un seguimiento de todos los recursos asignados en una variable global `resourceCount`.

**Impremiendo `Rx.resourceCount` después de empujar a un controlador de vista a la pila de navegación, usarlo, y despues volver saliendo con pop de este es generalmente la mejor manera de detectar y depurar fugas de recursos.**

Como prueba de cordura, sólo puede hacer un `print` en su controlador de vista` deinit` método.

El código sería algo como esto.

Como una comprobación sana, puede hacer un `print` en el método `deinit` de su controlador de vista.

El código sería algo como esto.

```swift
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

La razón por la que debe utilizar un pequeño retraso se debe a que a veces se necesita una pequeña cantidad de tiempo para que las entidades planificadoras para liberar su memoria.

## Variables

`Variable`s representan un estado observable. `Variable` sin contener el valor no puede existir porque inicializador requiere valor inicial.

Variable envuelve un [`Subject`](http://reactivex.io/documentation/subject.html). En concreto se trata de un `BehaviorSubject`. A diferencia de `BehaviorSubject`, sólo expone la interfaz `value`, por lo que nunca podrá terminar o fallar.

También emitirá su valor actual de inmediato en todos sus suscriptores.

```swift
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

imprimirá

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

## KVO 

KVO es un mecanismo de Objective-C. Eso significa que no se construyó con tipo seguro en mente. Este proyecto trata de resolver algunos de los problemas.

Hay dos maneras incorporadas para que esta biblioteca sea compatible con KVO.

```swift
// KVO
extension NSObject {
    public func rx_observe<Element>(keyPath: String, retainSelf: Bool = true) -> Observable<Element?> {}
}

#if !DISABLE_SWIZZLING
// KVO
extension NSObject {
    public func rx_observeWeakly<Element>(keyPath: String) -> Observable<Element?> {}
}
#endif
```

** Si compilador Swift no tiene una forma de inferir el tipo observado (devolver tipo observable), informará de error acerca de la función no existente. **

Aquí hay algunas maneras que usted le puede dar pistas sobre el tipo observado:

```swift
view.rx_observe("frame") as Observable<CGRect?>
```

o

```swift
view.rx_observe("frame")
    .map { (rect: CGRect?) in
        //
    }
```

### `rx_observe`

`rx_observe` es más eficiente porque es sólo un simple envoltorio alrededor del mecanismo KVO, pero tiene escenarios de uso más limitados

* Que puede ser utilizado para observar rutas a partir de `self` o de ancestros en el gráfico de propiedad (`retainSelf = false`)
* Que puede ser utilizado para observar rutas a partir de descendientes en el gráfico de propiedad (`retainSelf = true`)
* Los caminos tienen que consistir sólo en propiedades `strong`, de lo contrario se arriesga tumbar el sistema al no anular el registro observador KVO antes de dealloc.

Por ejemplo

```swift
self.rx_observe("view.frame", retainSelf = false) as Observable<CGRect?>
```

### `rx_observeWeakly`

`rx_observeWeakly` tiene algo de peor rendimiento, ya que tiene que manejar la desasignación de objetos de la memoria en el caso de las referencias débiles (`weak`).

Se puede utilizar en todos los casos donde `rx_observe` se pueden utilizar y, además:

* porque no retendrá objetivo observado, que puede ser utilizado para observar el gráfico de objeto arbitrario cuya relación de titularidad es desconocida
* puede ser utilizado para observar propiedades `weak`

Por ejemplo

```swift
someSuspiciousViewController.rx_observeWeakly("behavingOk") as Observable<Bool?>
```

### La observación de estructuras

KVO es un mecanismo de Objective-C por lo que depende en gran medida de `NSValue`. RxCocoa tiene especializaciones adicionales para `CGRect`, `CGPoint` y `CGSize` que hacen que sea conveniente para observar esos tipos.

Al observar algunas otras estructuras es necesario extraer las estructuras del `NSValue` manualmente, o crear sus propias especializaciones de secuencia observable.

[Aquí](https://github.com/ReactiveX/RxSwift/blob/master/RxCocoa/Common/Observables/NSObject%2BRx%2BCoreGraphics.swift) son ejemplos de cómo extraer correctamente las estructuras de `NSValue`.

## Consejos de la capa de interfaz de usuario

Hay ciertas cosas que sus `Observable`s necesidan satisfacer en la capa de interfaz de usuario cuando se une a los controles UIKit.

### Trabajando con hilos

Los `Observable`s necesitaran enviar valores en el `MainScheduler`(UIThread). Eso es sólo un requerimiento normal de UIKit/Cocoa.

Por lo general, es una buena idea para los APIs devuelvan los resultados en `MainScheduler`. En caso de que intenta enlazar algo al interfaz de usuario desde un hilo de segundo plano, construllendo en modo **debug** RxCocoa suele lanzar una excepción para informarle de ello.

Para solucionar este problema es necesario agregar `observeOn(MainScheduler.sharedInstance)`.

**Las extensiones de NSURLSession no vuelven resultado de `MainScheduler` por defecto.**

### Errores

No se puede enlazar el fallo a los controles UIKit porque eso es un comportamiento indefinido.

Si usted no sabe si `Observable` puede fallar, puede asegurarse de que no pueda hacerlo usando `catchErrorJustReturn(valorQueseRetornaCuandoOcurraUnError)`, **pero después de que ocurrá un error de la secuencia subyacente todavía completara**.

Si lo que se desea es que la secuencia subyacente siga produciendo elementos, se necesita alguna versión del operador `retry`.

### Compartiendo suscripciones

Generalmente, usted desea compartir la suscripción en la capa de interfaz de usuario. Usted no quiere hacer llamadas HTTP separadas para enlazar los mismos datos a múltiples elementos de la IU.

Digamos que usted tiene algo como esto:

```swift
let searchResults = searchText
    .throttle(0.3, $.mainScheduler)
    .distinctUntilChanged
    .map { query in
        API.getSearchResults(query)
            .retry(3)
            .startWith([]) // limpia los resultados en cada nuevo término de búsqueda
            .catchErrorJustReturn([])
    }
    .switchLatest()
    .shareReplay(1)        // <- fijese en el operador `shareReplay`
```

Lo que normalmente se quiere es compartir los resultados de búsqueda una vez calculados. Eso es lo que significa `shareReplay`.

** Por lo general, es una buena regla de oro en la capa de interfaz de usuario para añadir `shareReplay` al final de la cadena de transformación porque realmente desea compartir los resultados calculados. Usted no quiere disparar conexiones HTTP separadas al enlazar `searchResults` a varios elementos de la IU.**

## Hacer peticiones HTTP

Hacer peticiones http es una de las primeras cosas que la gente trata de hacer.

Primero tiene que construir un objeto `NSURLRequest` que representa el trabajo que hay que hacer.

Esta solicitud determina si es GET o POST, el cuerpo de la petición, los parámetros de consulta ...

Así es como se puede crear una solicitud GET sencilla

```swift
let request = NSURLRequest(URL: NSURL(string: "http://en.wikipedia.org/w/api.php?action=parse&page=Pizza&format=json")!)
```

Si se desea simplemente ejecutar esa petición fuera de la composición con otras observables, esto es lo que hay que hacer.

```swift
let responseJSON = NSURLSession.sharedSession().rx_JSON(request)

// hasta este momento no se ha llevado a caboninguna solicitud
// `responseJSON` is unicamente una description de como se extrae la respuesta

let cancelRequest = responseJSON
    // esto disparará la solicitud
    .subscribeNext { json in
        print(json)
    }

NSThread.sleepForTimeInterval(3)

// si se desea cancelar la solicitud después de pasen 3 segundos simplemente hay que llamar a
cancelRequest.dispose()
```

**Las extensiones de NSURLSession no vuelven resultado de `MainScheduler` por defecto.**

En caso de que desee un acceso de más bajo nivel a la respuesta, puede utilizar:

```swift
NSURLSession.sharedSession().rx_response(myNSURLRequest)
    .debug("my request") // esto va a imprimir la información a la consola
    .flatMap { (data: NSData!, response: NSURLResponse!) -> Observable<String> in
        if let response = response as? NSHTTPURLResponse {
            if 200 ..< 300 ~= response.statusCode {
                return just(transform(data))
            }
            else {
                return failWith(yourNSError)
            }
        }
        else {
            rxFatalError("response = nil")
            return failWith(yourNSError)
        }
    }
    .subscribe { event in
        print(event) // si ocurre un error, esto también se imprimirá el error a la consola
    }
```
### Registro de tráfico HTTP

En el modo de depuración RxCocoa registrará toda solicitud HTTP a la consola por defecto. En caso de que quiera cambiar ese comportamiento, configure el filtro `Logging.URLRequests`.

```swift
// lee nuetra propia configuración
public struct Logging {
    public typealias LogURLRequest = (NSURLRequest) -> Bool

    public static var URLRequests: LogURLRequest =  { _ in
    #if DEBUG
        return true
    #else
        return false
    #endif
    }
}
```

## RxDataSourceStarterKit

... es un conjunto de clases que implementan las fuentes de datos reactivas plenamente funcionales para `UITableView`s y `UICollectionView`s.

El código fuente, más información y razón de por qué estas clases se separan en su directorio se pueden encontrar [aquí](https://github.com/ReactiveX/RxSwift/tree/master/RxDataSourceStarterKit).

Su uso apenas debera importar todos los archivos en su proyecto.

Una demostración completamente funcional cómo usarlos se incluye en el proyecto [RxExample](https://github.com/ReactiveX/RxSwift/tree/master/RxExample).

