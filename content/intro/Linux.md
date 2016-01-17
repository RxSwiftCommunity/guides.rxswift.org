+++
date = "2015-10-23T16:12:42+02:00"
title = "Linux"
categories = "introducción"
tags = ["Linux", "guia", "operators", "threads", "documentación"]
+++

Linux
=====

Hemos hecho una prueba de concepto para Linux.

Para probarlo, crear `Package.swift` en su directorio prueba con el siguiente contenido:

```
import PackageDescription

let package = Package(
    name: "MyShinyUnicornCat",
    dependencies: [
        .Package(url: "https://github.com/ReactiveX/RxSwift.git", Version(2, 0, 0))
    ]
)
```

Lo que funciona:
* Distribución utilizando Swift Package Manager
* Modo único subproceso (CurrentThreadScheduler)
* La mitad de las pruebas unitarias están pasando.
* Los proyectos que se pueden compilar y "utilizar":
     * RxSwift
     * RxBlocking
     * RxTests

Lo que no funciona:
* Schedulers - porque son dependientes de https://github.com/apple/swift-corelibs-libdispatch y todavía no ha sido liberado
* Multithreading - todavía no tienen acceso a las cerraduras c11
* Por alguna razón parece que Swift compilador genera código incorrecto cuando se utiliza `ErrorType` sobre `Linux`, así que no use los errores, de lo contrario usted puede conseguir accidentes extraños.
