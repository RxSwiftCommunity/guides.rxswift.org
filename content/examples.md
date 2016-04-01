+++
date = "2015-10-23T17:38:00+02:00"
title = "Examples"
categories = "Examples"
tags = ["examples", "guide"]
+++

## 계산된 변수(Calculated variable)

RxSwift 에 대해 알아보기 전에, 우선 명령형(imperative) 방식의 Swift 코드에서 시작해보겠습니다.
이 예제의 목적은 특정 조건이 만족했을 때 `a` 와 `b` 로부터 계산된 값을 `c`에 반영(bind)하는 것입니다.

다음은 `c`의 값을 계산하는 명령형 swift 코드입니다.:

```swift
// 일반적인 명령형 코드입니다.
var c: String
var a = 1       // 이 구문은 `a`에 `1`을 단 한번 할당할 것입니다.
var b = 2       // 이 구문은 `b`에 `2`를 단 한번 할당할 것입니다.

if a + b >= 0 {
    c = "\(a + b) is positive" // 이 구문은 `c`에 값을 단 한번만 할당할 것입니다.
}
```

현재 `c`의 값은 `3 is positive`입니다. `a` 를 `4`로 변경해도, `c`의 값은 변하지 않을 것입니다.

```swift
a = 4           // c 가 여전히 이전 값인 "3 is positive"가 되는 것은 우리가 원하는 동작 방식이 아닐 것입니다.
                // 4 + 2 = 6 이므로 c 는 "6 is positive" 가 되어야 합니다.
```

이것은 우리가 원하는 동작 방식이 아닙니다.

프로젝트에서 RxSwift 프레임워크를 사용하고 싶다면 RxSwift 프레임워크를 프로젝트안에 포함시킨 다음, `import RxSwift` 를 선언하면 됩니다.

위의 예제를 RxSwift 로 작성하면 다음과 같습니다.

```swift
let a /*: Observable<Int>*/ = Variable(1)   // a = 1
let b /*: Observable<Int>*/ = Variable(2)   // b = 2

// RxSwift 코드는 rx 변수 `c` 에 아래의 주석처리된 로직을 반영할 것입니다.
// if a + b >= 0 {
//      c = "\(a + b) is positive"
// }
// `a`, `b` 변수의 최근 값을 `+`을 사용하여 더합니다.
let c = Observable.combineLatest(a.asObservable(), b.asObservable()) { $0 + $1 }     
  .filter { $0 >= 0 }               // 만약 `a + b >= 0` 가 `참`이라면, `a + b` 는 map 연산자에게 전달됩니다.
  .map { "\($0) is positive" }      // `a + b` 는 "\(a + b) is positive" 로 변환(mapping) 됩니다.

// 초기값이 a = 1, b = 2 이기 때문에,
// 1 + 2 = 3 은 >= 0 이며, `c` 는 "3 is positive" 라는 값을 갖게 됩니다.

// rx 변수인 `c` 에서 값을 얻기 위해, `c` 로부터 값을 subscribe 합니다.
// `subscribeNext` 는 `c` 변수에 새롭게 설정되는 값(next value)을 subscribe 합니다.
// 여기에는 초기값인 "3 is positive" 역시 포함됩니다.
c.subscribeNext { print($0) }          // prints: "3 is positive"

// `a` 의 값을 증가시켜 보겠습니다.
// RxSwift 에서는 a = 4 를 아래와 같이 표현합니다.
a.value = 4                                   // prints: 6 is positive
// 최근 값들의 합은 `4 + 2` 이며, `6` 은 >= 0 이기 때문에,
// map 연산자는 "6 is positive" 를 결과값으로 생성합니다.
// 그리고 그 결과값은 `c` 에 "할당됩니다".
// `c` 의 값이 변했기 때문에, `{ print($0) }` 이 호출될 것이며,
// "6 is positive" 가 화면에 출력될 것입니다.

// 이제 `b`의 값을 변경해보겠습니다.
// RxSwift 에서 b = -8 은 아래와 같이 나타낼 수 있습니다.
b.value = -8                                 // 화면에 아무 것도 출력되지 않습니다.
// 최근 값의 합인 `4 + (-8)` 은 `-4` 이며, 이것은 >= 0 가 아니므로,
// map 연산자가 실행되지 않습니다.
// 이는 `c` 는 여전히 "6 is positive" 라는 것을 의미합니다.
// `c` 가 변하지 않았기 때문에 다음 값(next value) 이 생성되지 않았으며,
// `{ print($0) }` 은 호출되지 않습니다.

// ...
```

## 간단한 UI 바인딩(bindings)

* 변수와 바인딩하는 대신 text field 값 (rx_text)과 바인딩합니다.
* 그 다음, 입력값을 int 로 파싱하여 그 숫자가 prime number 인지 아닌지를 비동기 API 를 사용하여 계산합니다.(map)
* 만약 text field 값이 비동기 호출이 완료되기 전에 변한다면, 다음 비동기 호출은 큐에 쌓습니다. (concat)
* 결과값을 label 에 바인딩합니다 (bindTo(resultLabel.rx_text))

```swift
let subscription/*: Disposable */ = primeTextField.rx_text      // type is Observable<String>
            .map { WolframAlphaIsPrime(Int($0) ?? 0) }       // type is Observable<Observable<Prime>>
            .concat()                                           // type is Observable<Prime>
            .map { "number \($0.n) is prime? \($0.isPrime)" }   // type is Observable<String>
            .bindTo(resultLabel.rx_text)                        

// 서버 호출이 완료된 후 resultLabel.text 는 "number 43 is prime? true" 으로 설정될 것입니다.
primeTextField.text = "43"

// ...

// 모든 것을 unbind 하기 위해 아래 코드를 호출합니다.
subscription.dispose()
```

특별할 것이 없습니다. 이번 예제에 사용된 모든 연산자들은 첫 번째 예제에서 사용된 것과 동일한 연산자들입니다.

## 자동 완성

만약 Rx 를 처음 접하는 것이라면, 다음 예제는 약간 놀라울 수 있습니다. 이번 예제는 RxSwift 코드가 실제로 어떻게 사용되는지 잘 보여주고 있습니다.

세번째 예제는 실제 개발에서 자주 경험하게 되는 비동기 적합성 확인(validation) 로직을 처리하는 복잡한 UI 를 다루고 있습니다. UI 는 진행상황을 알려주는 기능이 포함되어 있습니다.
모든 연산은 `disposeBag` 이 deallocate 되는 순간 취소됩니다.

예제를 살펴보겠습니다.

```swift
// UI control 값을 직접 바인딩합니다.
// `usernameOutlet` 으로부터 username 을 얻습니다.
self.usernameOutlet.rx_text
    .map { username in

        // 동기적인 확인(synchronous validation)을 진행합니다. 이곳에서는 특별한 것이 없습니다.
        if username.isEmpty {
            // 여기서는 동기, 비동기 코드가 같은 method 안에 혼합되어 있으며,
            // 이것은 바로 resolve 되는 비동기 결과값을 생성할 것입니다.
            return Observable.just((valid: false, message: "Username can't be empty."))
        }

        ...

        // 비동기 연산이 처리되는 동안, 모든 사용자 인터페이스는 현재의 진행 상황을 사용자에게 알려줍니다.
        // 결과를 기다리는 동안 사용자에게 "Checking availability" 라는 메시지를 보여준다고 가정해봅시다.
        // 적합성 판단에 사용되는 인자들은 다음과 같을 것입니다.
        //  * true  - 적합
        //  * false - 부적함
        //  * nil   - 적합성 판정 대기
        typealias LoadingInfo = (valid : String?, message: String?)
        let loadingValue : LoadingInfo = (valid: nil, message: "Checking availability ...")

        // 이것은 username 이 이미 존재하는지 확인하는 서버 요청을 실행할 것입니다.
        // 타입은 `Observable<ValidationResult>` 과 같은 형태일 것입니다.
        return API.usernameAvailable(username)
          .map { available in
              if available {
                  return (true, "Username available")
              }
              else {
                  return (false, "Username already taken")
              }
          }
          // 서버 반응이 도착할 때까지는 `loadingValue` 를 사용합니다.
          .startWith(loadingValue)
    }
// 현재 `Observable<Observable<ValidationResult>>` 을 가지고 있기 때문에
// `Observable` 세상으로 돌아갈 필요가 있습니다.
// 우리는 두번째 예제에서처럼 `concat` 연산자를 사용할 수 있지만,
// 그 대신, 새로운 username 이 제공된다면 대기 중인(pending) 비동기 연산을 취소하도록 만들려고 합니다.
// 이러한 역할은 `switchLatest` 가 담당합니다.
    .switchLatest()
// 이제 우리는 사용자 인터페이스와 바인딩할 필요가 있습니다.
// `Observable` 체인의 마지막에서 `subscribeNext` 가 바인딩을 처리할 수 있습니다.
// 이것은 모든 것을 unbind 하고 대기 중인(pending) 비동기 연산을 취소할 수 있는
// `Disposable` 객체를 생성할 것입니다.
    .subscribeNext { valid in
        errorLabel.textColor = validationColor(valid)
        errorLabel.text = valid.message
    }
// 수동으로 처리하는 대신
// 아래 코드를 추가하여 view controller dealloc 에서 모든 것을 자동적으로 dispose 할 수 있게 합니다.
    .addDisposableTo(disposeBag)
```

이보다 더 간단할 수 있을까요? [추가 예제들이](../RxExample) 저장소에 있으니 확인해보시길 바랍니다.

추가 예제들은 MVVM 패턴을 이용하거나 MVVM 패턴을 사용하지 않고 RxSwift를 어떻게 사용하는지 잘 보여주고 있습니다.
