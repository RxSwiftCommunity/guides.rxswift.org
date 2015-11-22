+++
date = "2015-10-23T17:38:08+02:00"
title = "Getting Started"
categories = "Comenzando"
tags = ["cocoapods", "carthage", "projecto", "xcode"]
+++

Para empezar con RxSwift y RxCocoa el proceso es muy simple, Rx no contiene dependencias externas.

Estas son las opciones soportadas actualmente:

### Manual

Abrir Rx.xcworkspace, elejir el equema `RxExample` y ejecutar. Este método construirá todo y ejecutará la aplicación de ejemplo

### [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

**¡IMPORTANTE! Para el apoyo de CocoaPods a TVOS se requiere la versión `0.39` o posterior.**

```
# Podfile
use_frameworks!

pod 'RxSwift', '~> 2.0.0-beta.3'
pod 'RxCocoa', '~> 2.0.0-beta.3'
pod 'RxBlocking', '~> 2.0.0-beta.3'
```

En el directorio del `Podfile` escribir:

```
$ pod install
```

### [Carthage](https://github.com/Carthage/Carthage)

**Para Xcode 7.0**

Agregar esto a `Cartfile`

```
git "git@github.com:ReactiveX/RxSwift.git" "2.0.0-beta.3"
```

En el directorio del `Cartfile` escribir:

```
$ carthage update
```

**Para Xcode 7.1 y soporte tvOS**

Add this to `Cartfile`

```
git "git@github.com:ReactiveX/RxSwift.git" "master-7.1"
```

En el directorio del `Cartfile` escribir:

```
$ carthage update
```

### Manually using git submodules

* Añadir RxSwift como un submódulo Git

En el directorio de su proyecto escribir:

```
$ git submodule add git@github.com:ReactiveX/RxSwift.git
```

Entonces:

* Arrastre `Rx.xcodeproj` en el navegador de su proyecto
* Vaya a `Project > Targets > Build Phases > Link Binary With Libraries`, haga clic en el `+` y seleccione los destinos `RxSwift-[Platform]` y `RxCocoa-[Platform]`

### iOS 7

iOS 7 es poco difícil, pero se puede hacer. El problema principal es que iOS 7 no admite los marcos dinámicos.

Estos son los pasos para incluir proyectos RxSwift/RxCocoa en un proyecto iOS7

* Los proyectos RxSwift/RxCocoa no tienen dependencias externas por lo que unicamente **incluyendo manualmente todos los archivos `.swift`, `.m` y `.h`** 
en el objetivo de construcción debe importar todo el código fuente necesario.

Usted puede hacer que al copiar los archivos manualmente o utilizando submódulos git.

`git submodule add git@github.com:ReactiveX/RxSwift.git`

Después de que hayan incluido los archivos de los directorios `RxCocoa` y `RxSwift`, se tendrá que eliminar los archivos que son específicos de otra plataforma.

* Si está compilando para `iOS`, por favor elimine las referencias a archivos específicos de **OSX** ubicadas en **`RxCocoa/OSX`**.
* Si está compilando para `OSX`, por favor elimine las referencias a archivos específicos de **iOS** ubicadas en **`RxCocoa/iOS`**.

Añadir **`RX_NO_MODULE`** como bandera personalizada de preprocesador personalizado Swift

Vaya a `Build Settings > Swift Compiler - Custom Flags` del objetivo de compilación y añada `-D RX_NO_MODULE`

Incluya **`RxCocoa.h`** en la cabecera de puente (bridging header)

Si usted ya tiene una cabecera de puente, añadiendole `#import "RxCocoa.h"` debería ser suficiente.

If you don't have a bridging header, you can go to your target's `Build Settings > Swift Compiler - Code Generation > Objective-C Bridging Header` and point it to `[path to RxCocoa.h parent directory]/RxCocoa.h`.

Si usted no tiene una cabecera de puente, se puede ir a `Build Settings > Swift Compiler - Code Generation > Objective-C Bridging Header` de tu objetivo y editar para que apunte a `[ruta de acceso a directorio padre RxCocoa.h]/RxCocoa.h`.
