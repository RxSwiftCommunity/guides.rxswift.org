+++
date = "2015-10-23T16:12:42+02:00"
title = "Unidades (Units)"
categories = "introducción"
tags = ["observables", "guide", "API", "documentation"]
+++

Este documento tratará de describir lo que son las unidades (Units), y porque son un concepto útil, cómo usarlos y cómo crearlos.

* [¿Por qué?](#por-qué)
* [Justificación de diseño](#design-rationale)
* ...

# ¿Por qué?

El propósito de las unidades es utilizar la comprobación de tipos estáticos del compilador de Swift para probar que su código se comporta como se ha diseñado.

Proyecto RxCocoa ya contiene varias unidades, pero el más elaborado se llama `Driver`, por lo que esta unidad se utiliza para explicar la idea detrás de unidades.

`Driver` fue nombrado de esa manera porque describe secuencias que conducen ciertas partes de la aplicación. Esas secuencias suelen manejar enlaces a la interfaz de usuario , los eventos de interfaz de usuario no solo mantienen su aplicación reactiva, sino que también maneja los servicios de la aplicación, etc.

El propósito de la unidad `Driver` es asegurar que la secuencia observable subyacente tiene las siguientes propiedades.

* No puede fallar, todos los fallos serán manejados adecuadamente
* Elementos se entregarán siempre por medio del hilo principal
* El cálculo de recursos de la secuencia se comparte

Por decidir...
