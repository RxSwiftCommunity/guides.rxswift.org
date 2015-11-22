+++
date = "2015-10-23T16:12:42+02:00"
title = "Observables frios y calientes"
categories = "introduction"
tags = ["observables", "guide", "documentation"]
+++


En mi humilde opinión, me permito sugerir pensar en esto más como ina propiedad de las secuencias que como tipos separados, ya que están representados por la misma abstracción que les encaja a la perfección, `secuencia Observable`.

Esta es una definición de ReactiveX.io

> ¿Cuándo comienza un observable que emite su secuencia de elementos? Depende del observable. Un observable "caliente" puede comenzar a emitir ítems tan pronto como se crea, por lo que cualquier observador que más tarde se suscriba al observable puede comenzar a observar la secuencia en algún punto intermedio. Un observable "frío", por otro lado, espera hasta que un observador se suscriba a el antes de que comienze a emitir ítems, y de esta manera está garantizado que el observador vea toda la secuencia desde el principio.

| Observables calientes                                                                                                                                      | Observables frios                                                                   |
|------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| ... son secuencias                                                                                                                                         | ... son secuencias                                                                  |
| Utilizar recursos ("produce calor"), no importa si tiene algun observador suscrito.                                                                        | No utiliza recursos (no produce calor) hasta que algun observador se suscriba.      |
| Variables / propiedades / constantes, coordenadas de un toque, coordenadas de click de ratón, valores de un control de interfaz de usuario, la hora actual | Operaciones asíncronas, conexiones HTTP, conexiones TCP, streams                    |
| Por lo general, contiene ~ N elementos                                                                                                                     | Por lo general, contiene 1 ~ elemento                                               |
| Los elementos de secuencia se producen sin importar si hay algun observador suscrito.                                                                      | Los elementos de secuencia se producen únicamente si hay algún observador suscrito. |
| Sequence computation resources are usually shared between all of the subscribed observers.                                                                 | Sequence computation resources are usually allocated per subscribed observer.       |
| Por lo general, con estado                                                                                                                                 | Por lo general, sin estado                                                          |
