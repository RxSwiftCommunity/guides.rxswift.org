+++
date = "2015-10-23T17:38:08+02:00"
title = "시작하기"
categories = "Introduction"
tags = ["cocoapods", "carthage", "project", "xcode"]
+++

RxSwift 와 RxCocoa 와 함께 시작하는 절차는 매우 간단합니다. Rx는 어떠한 외부 의존성도 필요하지 않습니다.

RxSwift 와 RxCocoa를 이용하기 위한 방법은 다음과 같습니다. :

### 설치방법

[RxSwift 프로젝트](https://github.com/ReactiveX/RxSwift)에서 제공하는 예제 파일을 다운받은 후, Rx.xcworkspace 을 실행하세요. 타겟에서 `RxExample` 을 선택한 후 프로그램을 실행하면 예제 앱을 확인할 수 있습니다.

### [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

**주의사항! tvOS를 지원하기 위해서는 CocoaPods `0.39` 버전 이상이 필요합니다.**

```
# Podfile
use_frameworks!

pod 'RxSwift', '~> 2.0.0-beta'
pod 'RxCocoa', '~> 2.0.0-beta'
pod 'RxBlocking', '~> 2.0.0-beta'
```

`Podfile` 파일을 위와 같이 작성한 다음, 아래 명령어를 이용해 pod를 설치하세요.

```
$ pod install
```

### [Carthage](https://github.com/Carthage/Carthage)

**Xcode 7.0인 경우**

`Cartfile` 파일에 아래의 구문을 추가하세요.

```
git "git@github.com:ReactiveX/RxSwift.git" "2.0.0-beta.1"
```

```
$ carthage update
```

**Xcode 7.1 이상 및 tvOS 지원**

`Cartfile` 파일에 아래의 구문을 추가하세요.

```
git "git@github.com:ReactiveX/RxSwift.git" "master-7.1"
```

```
$ carthage update
```

### git 서브모듈을 이용한 설치

* RxSwift를 submodule로 추가하세요.

```
$ git submodule add git@github.com:ReactiveX/RxSwift.git
```

* 프로젝트 네비게이터로 `Rx.xcodeproj`를 드래그합니다.
* `Project > Targets > Build Phases > Link Binary With Libraries`에서 `+`를 클릭하고 `RxSwift-[Platform]`, `RxCocoa-[Platform]` 타겟을 선택합니다.

### iOS 7

iOS 7의 경우 조금 까다롭습니다. iOS7의 가장 큰 문제점은 dynamic frameworks를 지원하지 않는다는 점입니다.

iOS7 프로젝트에 RxSwift/RxCocoa 프로젝트를 포함시키기 위한 방법은 다음과 같습니다.

* RxSwift/RxCocoa 프로젝트는 외부 의존성이 없기 때문에 build target 에 **모든 `.swift`, `.m`, `.h` 파일들** 을 포함시키면 필요한 소스 코드는 모두 import 됩니다.

이를 위해 직접 파일들을 복사하거나 git 서브모듈을 이용할 수 있습니다.

`git submodule add git@github.com:ReactiveX/RxSwift.git`

`RxSwift` and `RxCocoa` 디렉토리로부터 파일을 추가한 뒤에는, 특정 플랫폼과 관련된 파일들을 삭제해야 합니다.

`iOS`을 대상으로 개발하는 경우, **`RxCocoa/OSX`** 에 위치한 OSX 관련 파일들을 **삭제(remove reference)합니다.**.

`OSX`을 대상으로 개발하는 경우, **`RxCocoa/iOS`** 에 위치한 iOS 관련 파일들을 **삭제(remove reference)합니다.**.

* Swift preprocessor 플래그로 **`RX_NO_MODULE`** 을 추가합니다.

target의 `Build Settings > Swift Compiler - Custom Flags` 로 이동하여 `-D RX_NO_MODULE`를 추가합니다.

* bridging header에 **`RxCocoa.h`** 를 추가합니다.

이미 bridging header를 가지고 있는 경우, `#import "RxCocoa.h"` 를 추가하는 것만으로도 충분합니다.

만약 bridging header가 없는 경우, target의 `Build Settings > Swift Compiler - Code Generation > Objective-C Bridging Header` 로 이동하여  `[RxCocoa.h 부모 디렉토리 경로]/RxCocoa.h` 와 같이 경로를 추가합니다.
