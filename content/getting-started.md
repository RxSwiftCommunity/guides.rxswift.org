+++
date = "2015-10-23T17:38:08+02:00"
title = "Getting Started"
categories = "Introduction"
tags = ["cocoapods", "carthage", "project", "xcode"]
+++

To get started with RxSwift and RxCocoa the process is very simple, Rx doesn't contain any external dependencies.

These are currently supported options:

### Manual

Open Rx.xcworkspace, choose `RxExample` and hit run. This method will build everything and run sample app

### [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

**IMPORTANT! For tvOS support CocoaPods `0.39` is required.**

```
# Podfile
use_frameworks!

pod 'RxSwift', '~> 2.0.0-beta'
pod 'RxCocoa', '~> 2.0.0-beta'
pod 'RxBlocking', '~> 2.0.0-beta'
```

type in `Podfile` directory

```
$ pod install
```

### [Carthage](https://github.com/Carthage/Carthage)

**For Xcode 7.0**

Add this to `Cartfile`

```
git "git@github.com:ReactiveX/RxSwift.git" "2.0.0-beta.1"
```

```
$ carthage update
```

**For Xcode 7.1 and tvOS support**

Add this to `Cartfile`

```
git "git@github.com:ReactiveX/RxSwift.git" "master-7.1"
```

```
$ carthage update
```

### Manually using git submodules

* Add RxSwift as a submodule

```
$ git submodule add git@github.com:ReactiveX/RxSwift.git
```

* Drag `Rx.xcodeproj` into Project Navigator
* Go to `Project > Targets > Build Phases > Link Binary With Libraries`, click `+` and select `RxSwift-[Platform]` and `RxCocoa-[Platform]` targets

### iOS 7

iOS 7 is little tricky, but it can be done. The main problem is that iOS 7 doesn't support dynamic frameworks.

These are the steps to include RxSwift/RxCocoa projects in an iOS7 project

* RxSwift/RxCocoa projects have no external dependencies so just manually **including all of the `.swift`, `.m`, `.h` files** in build target should import all of the necessary source code.

You can either do that by copying the files manually or using git submodules.

`git submodule add git@github.com:ReactiveX/RxSwift.git`

After you've included files from `RxSwift` and `RxCocoa` directories, you'll need to remove files that are platform specific.

If you are compiling for `iOS`, please **remove references** to OSX specific files located in **`RxCocoa/OSX`**.

If you are compiling for `OSX`, please **remove references** to iOS specific files located in **`RxCocoa/iOS`**.

* Add **`RX_NO_MODULE`** as a custom Swift preprocessor flag

Go to your target's `Build Settings > Swift Compiler - Custom Flags` and add `-D RX_NO_MODULE`

* Include **`RxCocoa.h`** in your bridging header

If you already have a bridging header, adding `#import "RxCocoa.h"` should be sufficient.

If you don't have a bridging header, you can go to your target's `Build Settings > Swift Compiler - Code Generation > Objective-C Bridging Header` and point it to `[path to RxCocoa.h parent directory]/RxCocoa.h`.
