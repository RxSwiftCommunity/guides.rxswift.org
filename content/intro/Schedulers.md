+++
date = "2015-10-23T16:12:42+02:00"
title = "Planificadores (Schedulers)"
categories = "introducción"
tags = ["Schedulers", "guide", "operators", "threads", "documentation"]
+++

1. [Planificadores Serie frente a Concurrentes](#serial-vs-concurrent-schedulers)
1. [Planificadores a medida](#custom-schedulers)
1. [Planificadores incorporados](#builtin-schedulers)


Los Planificadores nos abstraen lejos del mecanismo que realiza el trabajo.

Los diferentes mecanismos para la realización de trabajos incluyen, subproceso actual, colas de despacho, colas de operación, nuevos tareas, depósitos de tareas, ejecución de bucles...

Hay dos principales operadores que trabajan con los planificadores. `observeOn` y `subscribeOn`.

Si desea realizar trabajos en diferentes planificadores sólo tiene que utilizar el operador `observeOn`.

Normalmente se usará mucho más a menudo `observeOn` que `subscribeOn`.

En el caso de `observeOn` no se especifica de forma explícita, el trabajo se realizará en el subproceso/planificador que los elementos estén generando.

Ejemplo de uso de operador `observeOn`

```
sequence1
  .observeOn(planificadorDeSegundoPlano)
  .map { n in
      print("Esto se realizará en un planificador que ejecuta las tareas en segundo plano")
  }
  .observeOn(MainScheduler.instance)
  .map { n in
      print("Esto se realizará en el planificador principal")
  }
```

Si quieres empezar la generación de la secuencia usé el método `subscribe` y un planificador específico, utilice `subscribeOn(planificador)`.

En el caso de `subscribeOn` no se especifique explícitamente, el método `subscribe` será llamado en el mismo subproceso/planificador que se llame a `subscribeNext` o `subscribe`.

En el caso de `subscribeOn` no se especifique explícitamente, se llamará al método `dispose` en el mismo subproceso/planificador que inicie la eliminación.

En resumen, si no se elige un planificador concreto, esos métodos serán llamados en el subproceso/planificador actual.

# Planificadores Serie frente a Concurrentes

Ya que los planificadores realmente puede ser cualquier cosa, y todos los operadores que transforman secuencias necesiten mantener [garantías implícitas](/intro/#implicit-observable-guarantees) adicionales, es importante qué tipo de planificador se está creando.

En caso de que el planificador es concurrente, los operadores Rx `observeOn` y `subscribeOn` será asegurarse de que todo funciona perfecto.

Si utiliza algún planificador que Rx pueda demostrar que es de serie, será capaz de realizar optimizaciones adicionales.

Hasta ahora sólo la realización de esas optimizaciones para planificadores de cola de envío.

En el caso de los planificadores de cola de despachado en serie `observeOn` estará optimizado con una simple llamada a `dispatch_async`.

# Planificadores a medida

Además de los planificadores incorporados, es posible escribir sus propios planificadores.

Si lo que desea es describir que se necesita para realizar el trabajo de inmediato, puede crear su propio planificador mediante la implementación del protocolo ImmediateScheduler`.

```swift
public protocol ImmediateScheduler {
    func schedule<StateType>(state: StateType, action: (/*ImmediateScheduler,*/ StateType) -> RxResult<Disposable>) -> RxResult<Disposable>
}
```

Si desea crear un nuevo planificador que soporte operaciones basadas en el tiempo, entonces usted tendrá que implementar.

```swift
public protocol Scheduler: ImmediateScheduler {
    typealias TimeInterval
    typealias Time

    var now : Time {
        get
    }

    func scheduleRelative<StateType>(state: StateType, dueTime: TimeInterval, action: (StateType) -> RxResult<Disposable>) -> RxResult<Disposable>
}
```

En caso de que el planificador solo tiene capacidades periódicas de planificación, puede informar a Rx implementando el protocolo `PeriodicScheduler`

```swift
public protocol PeriodicScheduler : Scheduler {
    func schedulePeriodic<StateType>(state: StateType, startAfter: TimeInterval, period: TimeInterval, action: (StateType) -> StateType) -> RxResult<Disposable>
}
```

En caso de que el planificador no admita capacidades de `PeriodicScheduling`, Rx emulará la planificación periódica de forma transparente.

# Planificadores incorporados

Rx puede utilizar todos los tipos de planificador, pero también puede realizar algunas optimizaciones adicionales si tiene prueba de que el planificador es serie.

Estos son programadores actualmente soportados:



## CurrentThreadScheduler (Planificador Serie)

Planifica las unidades de trabajo en el subproceso actual.
Este es el programador predeterminado para los operadores que generan elementos.

Este planificador también a veces se llama `trampoline scheduler` planificador trampolín.

Si se llama a `CurrentThreadScheduler.instance.schedule(state) { }` por primera vez en algún hilo, la acción programada se ejecutará inmediatamente y se creara una cola oculta en todas las acciones planificadas de forma recursiva se añadan a la cola temporalmente.

Si algún campo padre en la pila de llamada ya se está ejecutando `CurrentThreadScheduler.instance.schedule(state) { }`, la acción planificada se añadirá a la cola y ejecutada cuando la accion que se esta ejecutando actualmente y todas las acciones añadidas a la cola anteriormente hayan finalizado su ejecución.



## MainScheduler (Planificador Serie)

Abstrae el trabajo que debe llevarse a cabo en `MainThread`. En el caso de los métodos `schedule` se llaman desde el hilo principal, se llevará a cabo la acción de inmediato y sin planificación.

Este programador se utiliza generalmente para realizar un trabajo sobre la interfaz de usuario.



## SerialDispatchQueueScheduler (Planificador Serie)

Abstrae el trabajo que debe llevarse a cabo en un `dispatch_queue_t` específico. Se asegurará de que, aunque se le pase una cola de despachado concurrente, será transformado en uno serie.

Los planificadores serie permiten ciertas optimizaciones para `observeOn`.

Planificador principal `MainScheduler.sharedInstance` es una instancia de `SerialDispatchQueueScheduler`.



## ConcurrentDispatchQueueScheduler (Concurrent scheduler)

Abstrae el trabajo que debe llevarse a cabo en un `dispatch_queue_t` específico. También se puede pasar una cola de despachado serie, ya que no debería causar problemas.

Este planificador es adecuado cuando se necesita realizar algún trabajo en segundo plano.



## OperationQueueScheduler (Concurrent scheduler)

Abstrae el trabajo que hay que realizar en un `NSOperationQueue` específico.

Este planificador es adecuado para los casos donde haya que hacer gran cantidad de trabajo en y se deseé afinar el procesamiento concurrente usando `maxConcurrentOperationCount`.

