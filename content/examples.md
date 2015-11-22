+++
date = "2015-10-23T17:38:00+02:00"
title = "Ejemplos"
categories = "Ejemplos"
tags = ["ejemplos", "guia"]
+++

## Variable calculada

Primero vamos a empezar con algo de código Swift imperativo.

El propósito del ejemplo es de enlazar el identificador `c` con un valor calculado a partir de 'A' y 'B' si alguna condición se cumple.

Aquí está el código imperativo Swift que calcula el valor de `c`:

```swift
// esto es código imperativo habitual
var c: String
var a = 1       // esto sólo asignará el valor `1` a `a` una vez
var b = 2       // esto sólo asignará el valor `2` a `b` una vez

if a + b >= 0 {
    c = "\(a + b) es positivo" // esto sólo asignará el valor a `c` una vez
}
```

El valor de `c` es ahora `3 es positivo`. Pero si cambiamos el valor de 'a' a `4`, `c` aún contendrá el valor antiguo.

```swift
a = 4           // c seguirá siendo igual a "3 es positivo", que no es bueno
                // c deberia ser "6 es positivo" porque 4 + 2 = 6
```

Y este no es el comportamiento deseado.

### Esta es la misma lógica usando RxSwift.

Para integrar el marco RxSwift en su proyecto unicamente debe incluir el marco en su proyecto y escribir `import RxSwit` en cada archivo Swift que lo requiera.

```swift
let a /*: Observable<Int>*/ = Variable(1)   // a = 1
let b /*: Observable<Int>*/ = Variable(2)   // b = 2

// Esto "enlazará" la variable rx `c` a la siguiente definición
// if a + b >= 0 {
//      c = "\(a + b) is positive"
// }

let c = combineLatest(a, b) { $0 + $1 }     // combina los últimos valores de las variables 'a' y 'b' usando `+`
    .filter { $0 >= 0 }                     // Si `a + b >= 0` es verdadero, `a + b` es pasado al operador map
    .map { "\($0) es positivo" }            // mapea `a + b` a "\(a + b) es positivo"

// Dado que los valores iniciales son a = 1, b = 2
// 1 + 2 = 3 que es >= 0, `c` es inicialmete igual a "3 es positivo"

// Para extraer valores de la variable rx `c`, hay que suscribirse a los valores de `c`.
// `subscribeNext` significa suscribirse a los valores siguientes (recientes) de la variable `c`.
// Eso también incluye el valor inicial "3 es positivo".
c.subscribeNext { print($0) }          // imprime: "3 es positivo"

// Ahora vamos a incrementar el valor de `a`
// a = 4 en RxSwift es:
a.next(4)                                   // imprime: "6 es positivo"
// La suma de los últimos valores ahora es `4 + 2`, `6` es >= 0, operador map
// produce "6 es positivo" y este resultado es "asignado" a `c`.
// Dado que el valor de `c` ha cambiado, `{ print($0) }` sera llamado,
// y "6 es positivo" será impreso.

// Ahora vamos a cambiar el valor de `b`
// b = -8 en RxSwift es:
b.next(-8)                                  // no imprime nada
// La suma de los últimos valores es `4 + (-8)`, `-4` no es >= 0, por lo que map no sera ejecutado.
// Esto significa que `c` todavía contiene "6 es positivo" y eso es correcto.
// Dado que `c` no ha sido actualizado, significa que el siguiente valor no se ha producido,
// y `{ print($0) }` no se llamara.

// ...
```

## Simple enlace con la interfaz de usuario

* en lugar enlazar a variables, vamos a elazar a los valores de un campo de texto (rx_text)
* despues, lo covertimos en un entero `int` y calculamos si el número es primo mediante una API asíncrona (map)
* si el valor del campo de texto se cambia antes de que finalice la llamada asíncrona, una nueva llamada asíncrona estará en cola (concat)
* enlazar los resultados a la etiqueta `UILabel` (resultLabel.rx_subscribeTextTo)

```swift
let subscription/*: Disposable */ = primeTextField.rx_text      // hasta aquí es Observable<String>
            .map { WolframAlphaIsPrime($0.toInt() ?? 0) }       // hasta aquí es Observable<Observable<Prime>>
            .concat()                                           // hasta aquí es Observable<Prime>
            .map { "¿el número \($0.n) es primo? \($0.isPrime)" }   // hasta aquí es Observable<String>
            .bindTo(resultLabel.rx_text)                        // devuelve Disposable que se puede utilizar para desenlazar todo

// Después de que se complete la llamada al servidor se establecerá resultLabel.text a "¿el número 43 es primo? true".
primeTextField.text = "43"

// ...

// para desenlazar todo, simplemente llame a
subscription.dispose()
```

Todos los operadores utilizados en este ejemplo son los mismos operadores utilizados en el primer ejemplo con variables. No hay nada especial en ello.

## Autocompletar

Si eres nuevo en Rx, el siguiente ejemplo será probablemente un poco abrumador, pero está aquí para demostrar cómo el código RxSwift se ve como en ejemplos del mundo real.

El tercer ejemplo es un mundo real, lógica de validación asíncrona de interfaz de usuario compleja, con notificaciones de progreso.
Todas las operaciones se cancelaran en el momento que `disposeBag` se desasigne.

Vamos a darle una oportunidad.

```swift
// los valores de control de interfaz de usuario se enlazan directamente
// usa el nombre de usuario de `usernameOutlet` como el valor fuente `sername`
self.usernameOutlet.rx_text
    .map { username in

        // validación sincrónica, nada especial aquí
        if count(username) == 0 {
            // Conveniencia para la construcción de resultado síncronos.
            // En caso de que haya código sincrónico y asincrónico mixto dentro del mismo 
            // método, construirá un resultado asíncrono que se resuelve inmediatamente.
            return just((valid: false, message: "Nombre de usuario no puede estar vacío."))
        }

        ...

        // Cada interfaz de usuario, probablemente, muestra un estado, mientras que 
        // la operación asíncrona se está ejecutando.
        // Supongamos que queremos mostrar "Comprobando disponibilidad" mientra esperamos el resultado.
        // El parámetro válido puede ser:
        //  * true  - si es valido
        //  * false - si es invalido
        //  * nil   - si esta pendiente de validación
        let loadingValue = (valid: nil, message: "Comprobando disponibilidad...")

        // Esto se disparará una llamada servidor para comprobar si ya existe el nombre de usuario.
        // ¿Sabes una cosa? Su tipo es `Observable<ValidationResult>`
        return API.usernameAvailable(username)
          .map { available in
              if available {
                  return (true, "Nombre de usuario disponible")
              }
              else {
                  return (false, "Este nombre de usuario ya se esta usando")
              }
          }
          // usa `loadingValue` hasta que el servidor responda
          .startWith(loadingValue)
    }
// Puesto que ahora tenemos `Observable<Observable<ValidationResult>>`
// que de alguna manera tenemos que volver a la normalidad del mundo `Observable`.
// Podriamos usar el operador `concat` que utilizamos en el segundo ejemplo, pero lo que
// realmente queremos es cancelar las operaciones asincrónico pendientes si un nuevo nombre de 
// usuario es suministrado.
// Todo esto es lo que `switchLatest` hace
    .switchLatest()
// Ahora de alguna manera tenemos que enlazar esto a la interfaz de usuario.
// Nuestro viejo amigo `subscribeNext` puede hacerlo
// Este es el final de la cadena `Observable`.
// Esto producira un objeto `Disposable` (desechable) que puede desenlazar todo y cancelar
// las operaciones asincrónicas pendientes.
    .subscribeNext { valid in
        errorLabel.textColor = validationColor(valid)
        errorLabel.text = valid.message
    }
// ¿Por qué deberíamos hacer esto de forma manual?, es tedioso,
// vamos a desechar todo automágicamente cuando el controlador de vista se desasigne.
    .addDisposableTo(disposeBag)
```

No se puede obtener nada más sencillo que esto. Hay [más ejemplos](https://github.com/ReactiveX/RxSwift/tree/master/RxExample) en el repositorio, así que siéntete libre de echarle un vistazo.

Incluyen ejemplos de cómo usarlo en el contexto del patrón MVVM o sin él.

