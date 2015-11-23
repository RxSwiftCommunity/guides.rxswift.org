+++
date = "2015-10-23T16:12:42+02:00"
title = "Las matemáticas detrás de Rx"
categories = "introducción"
tags = ["guide", "documentation"]
+++

## La dualidad entre Observador y Iterator / Enumerador / Generador / Secuencias

Hay una dualidad entre el observador y el patrón generador. Eso es lo que permite la transición del mundo de devolución de llamada asincrónica al mundo de transformaciones de secuencia sincrónico.

En resumen, enumerador y el patrón de observador ambos describen secuencias. Es bastante obvio por qué lo hace enumerador secuencia definida, pero ¿qué hay del observador?.

También hay una explicación bastante simple que no incluye una gran cantidad de matemáticas. Suponga que está observando los movimientos del ratón. Cada movimiento del ratón recibido es un elemento de una secuencia de movimientos del ratón sobre el tiempo.

En resumen, hay dos maneras de acceder a los elementos básicos de una secuencia.

* Interfaz Push - Observadores (elementos observados que a lo largo del tiempo hacen una secuencia)
* Interfaz Pull - Iterator / Enumerador / Generador

Para aprender más acerca de esto, estos videos deben ayudar

También puede ver una explicación más formal explicó de una manera divertida en este video:

[Expertos para expertos: Brian Beckman y Erik Meijer - Dentro del Framework Reactivo de .NET (Rx) (vídeo)] (https://www.youtube.com/watch?v=looJcaeboBY) (en inglés)

