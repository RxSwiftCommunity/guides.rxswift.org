+++
date = "2015-10-23T16:12:42+02:00"
title = "Justificación de diseño"
categories = "introducción"
tags = ["error type", "Design", "Event", "guia", "API", "documentación"]
+++

## ¿Por qué el tipo de error no es genérico?

```swift
enum Event<Element>  {
    case Next(Element)      // el elemento siguiente de la secuencia
    case Error(ErrorType)   // la secuencia falla y produce un error
    case Completed          // la secuencia termina con éxito
}
```

Vamos a discutir los pros y contras de que `ErrorType` sea genérico.

Al tener un tipo de error genérico se crea una diferencia de impedancia adicional entre dos observables.

Digamos que tiene:

`Observable <String, E1>` y `observable <String, E2>`

No hay mucho que puedes hacer con ellos sin averiguar cuál será el tipo de error resultante.

¿Será `E1`,` E2` o tal vez algún nuevo `E3`? Por lo que se necesita un nuevo conjunto de operadores sólo para resolver el desajuste de impedancia.

Esto seguro que daña las propiedades de la composición, y Rx realmente no le importa qué secuencia falla, simplemente por lo general remite el fracaso bajo la cadena observable.

Existe un problema adicional que puede que en algunos casos los operadores fallen por algún error interno, y en ese caso usted no será capaz de construir error resultante e informar fracaso.

Pero bueno, vamos a ignorar eso y asumir que podemos usar eso para modelar secuencias que no lleven a cabo un error. Parece que podría ser útil para este propósito?

Bueno, sí, que potencialmente podría ser, pero consideremos por qué querrías usar secuencias que no lleven a cabo un error.

Una aplicación obvia sería para los flujos permanentes en la capa de interfaz de usuario que impulsan toda la interfaz de usuario. Pero si tenemos en cuenta ese caso, no es realmente sólo suficiente para utilizar el compilador de demostrar que las secuencias no error a cabo, también es necesario para probar otras propiedades. Al igual que los elementos se observan en `MainScheduler`.

Lo que realmente necesita es una manera genérica para demostrar rasgos de secuencias (`Observables`). Y usted podría estar interesado en una gran cantidad de propiedades. Por ejemplo:

* La secuencia termina en tiempo finito (en el lado del servidor)
* La secuencia contiene un solo elemento (si está ejecutando algún cálculo)
* La secuencia no lleva el error a cabo, nunca se termina y los elementos se entregan en el planificador principal `MainScheduler` (IU)
* La secuencia no lleva el error a cabo, nunca se termina y elementos están presentadas el planificador principal `MainScheduler`, y comparte cuentas de referencias (IU)
* La secuencia no lleva el error a cabo, nunca se termina y los elementos se entregan en planificador fondo específico (motor de audio)

Lo que realmente quiero es el compilador fuerce al sistema general de rasgos para las secuencias observables, y un conjunto de operadores invariables para esas propiedades deseadas.

Una buena analogía sería en mi humilde opinión

```
1, 3.14, e, 2.79, 1 + 1i      <->    Observable<E>
1m/s, 1T, 5kg, 1.3 pounds     <->    Errorless observable, UI observable, Finite observable ...
```

Hay muchas maneras de cómo hacer esto en Swift, ya sea usando la composición o la herencia de observables.

Un beneficio adicional de usar sistema de unidades es que se puede demostrar que el código de interfaz de usuario está ejecutando en el mismo planificador y así utilizar operadores sin bloqueo para todas las transformaciones.

Desde que Rx ya no tiene bloqueos para las operaciones de secuencias individuales, y todos los bloqueos restantes se encuentran en componentes sin estado (también llamado IU), sería prácticamente eliminar todos los bloqueos restantes de código de Rx y crear código Rx sin bloqueo que fuerce al compilador.

Así que en mi humilde opinión, realmente no hay beneficio del uso de Errores con tipo que no pueda lograrse de otras maneras mas limpias, preservando semántica composicional Rx. Y otras maneras también tienen otros beneficios enormes.

