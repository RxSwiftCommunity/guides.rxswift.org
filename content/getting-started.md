+++
date = "2015-10-23T17:38:08+02:00"
title = "Getting Started"
categories = "Comenzando"
tags = ["cocoapods", "carthage", "projecto", "xcode"]
+++

Para empezar con RxSwift y RxCocoa el proceso es muy simple, Rx no contiene dependencias externas.

Estas son las opciones soportadas actualmente:

### Manual

Abrir Rx.xcworkspace, elegir el esquema `RxExample` y ejecutar. Este método construirá todo y ejecutará la aplicación de ejemplo

### [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

**¡IMPORTANTE! Para el apoyo de CocoaPods a tvOS se requiere la versión `0.39` o posterior.**

```
# Podfile
use_frameworks!

pod 'RxSwift', '~> 2.1'
pod 'RxCocoa', '~> 2.1'
pod 'RxBlocking', '~> 2.1'
```

En el directorio del `Podfile` escribe:

```
$ pod install
```

### [Carthage](https://github.com/Carthage/Carthage)

**Para Xcode 7.1 o posterior**

Agrega esto al `Cartfile`

```
github "ReactiveX/RxSwift" "2.1"
```

En el directorio del `Cartfile` escribe:

```
$ carthage update
```

### Utilizando manualmente submódulos de git

* Añadir RxSwift como un submódulo Git

En el directorio de su proyecto escribe:

```
$ git submodule add git@github.com:ReactiveX/RxSwift.git
```

Entonces:

* Arrastre `Rx.xcodeproj` en el navegador de su proyecto
* Vaya a `Project > Targets > Build Phases > Link Binary With Libraries`, haga clic en el `+` y seleccione los destinos `RxSwift-[Platform]` y `RxCocoa-[Platform]`

### iOS 7

iOS 7 es poco difícil, pero se puede hacer. El problema principal es que iOS 7 no admite los marcos dinámicos.

Estos son los pasos para incluir proyectos RxSwift/RxCocoa en un proyecto iOS7

* Los proyectos RxSwift/RxCocoa no tienen dependencias externas por lo que únicamente **incluyendo manualmente todos los archivos `.swift`, `.m` y `.h`** 
en el objetivo de construcción debe importar todo el código fuente necesario.

Usted puede hacer que al copiar los archivos manualmente o utilizando submódulos git.

`git submodule add git@github.com:ReactiveX/RxSwift.git`

Después de que hayan incluido los archivos de los directorios `RxCocoa` y `RxSwift`, se tendrá que eliminar los archivos que son específicos de otra plataforma.

* Si está compilando para `iOS`, por favor elimine las referencias a archivos específicos de **OSX** ubicadas en **`RxCocoa/OSX`**.
* Si está compilando para `OSX`, por favor elimine las referencias a archivos específicos de **iOS** ubicadas en **`RxCocoa/iOS`**.

Añadir **`RX_NO_MODULE`** como bandera personalizada de preprocesador personalizado Swift

Vaya a `Build Settings > Swift Compiler - Custom Flags` del objetivo de compilación y añada `-D RX_NO_MODULE`

Incluya **`RxCocoa.h`** en la cabecera de puente (bridging header)

Si usted ya tiene una cabecera de puente, añadiéndole `#import "RxCocoa.h"` debería ser suficiente.

Si usted no tiene una cabecera de puente, se puede ir a `Build Settings > Swift Compiler - Code Generation > Objective-C Bridging Header` de tu objetivo y editar para que apunte a `[ruta de acceso a directorio padre RxCocoa.h]/RxCocoa.h`.
