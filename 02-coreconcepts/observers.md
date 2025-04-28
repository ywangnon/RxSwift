# Observers

## 👀 Observers — 스트림을 ‘관찰’하는 존재

> **Observer = 스트림 소비자**. Observable이 데이터를 "발행"(push)하면, Observer는 이를 "구독"(pull X)하여 처리합니다.

이 장에서는 RxSwift에서 Observer가 어떤 역할을 하며, 실제 프로젝트에서 어떻게 정의·활용되는지 살펴봅니다.

***

### 1️⃣ Observer의 기본 구조

```swift
public protocol ObserverType {
    associatedtype Element
    func on(_ event: Event<Element>)
}
```

* `Element`: 전달받을 데이터 타입 `<T>`
* `Event<Element>`: `.next(Element)`, `.error(Error)`, `.completed` 3종 중 하나

> 실제 앱에서는 대부분 **클로저 기반 구독**(`subscribe`)으로 Observer를 암묵적으로 생성하므로, 직접 프로토콜을 구현할 일은 드뭅니다.

***

### 2️⃣ subscribe 메서드 4총사

| 메서드 시그니처                     | 사용처              | 비고                        |
| ---------------------------- | ---------------- | ------------------------- |
| `subscribe(onNext:)`         | 값만 처리            | 주로 UI 바인딩 간단 처리           |
| `subscribe(onNext:onError:)` | 값 + 에러           | 네트워크·DB 등 실패 가능성          |
| `subscribe(on:)` (Full)      | `Event<T>` 직접 분기 | 고급 로깅·모니터링                |
| `bind(to:)`                  | UI 컴포넌트 바인딩      | 자동 MainScheduler, 에러 전달 X |

```swift
// 값만 필요할 때
button.rx.tap
    .subscribe(onNext: { print("Tapped!") })
    .disposed(by: bag)

// 에러도 캐치
fetchPosts()
    .subscribe(onNext: { print($0) },
               onError: { print("❌", $0) })
    .disposed(by: bag)
```

***

### 3️⃣ AnyObserver & Binder (커스텀 Observer)

#### 🎛️ AnyObserver

> 임의의 클로저를 Observer로 래핑할 때 사용

```swift
let labelObserver: AnyObserver<String> = AnyObserver { event in
    switch event {
    case .next(let text): label.text = text
    case .error(let err): print(err)
    case .completed: break
    }
}

Observable.just("Hello")
    .subscribe(labelObserver)
```

#### 🪄 Binder

> **UI 전용**. MainScheduler에서 실행되고 에러는 자동 무시(전파 안 함)

```swift
extension Reactive where Base: UILabel {
    var highlighted: Binder<Bool> {
        Binder(base) { label, active in
            label.isHighlighted = active
        }
    }
}

someBoolStream
    .bind(to: label.rx.highlighted)
    .disposed(by: bag)
```

***

### 4️⃣ Scheduler & Threading 주의

* **Observer가 실행되는 스레드**는 직전에 호출된 `observe(on:)` 영향.
* UI 업데이트는 반드시 **MainScheduler**에서!

```swift
imageDataStream
    .observe(on: MainScheduler.instance)
    .subscribe(onNext: imageView.setImage)
    .disposed(by: bag)
```

💡 `Driver`·`MainScheduler.instance` 사용으로 UI 스레드 보장 & 에러 무시 패턴이 인기.

***

### 5️⃣ Observer 패턴 실전 — 로딩 상태 관리

```swift
let loading = PublishSubject<Bool>()

loading
    .bind(to: activityIndicator.rx.isAnimating)
    .disposed(by: bag)

apiRequest()
    .do(onSubscribe: { loading.onNext(true) },
        onDispose:   { loading.onNext(false) })
    .subscribe()
    .disposed(by: bag)
```

> `do(onSubscribe:onDispose:)`는 **사이드 이펙트** 훅으로, 스트림 자체엔 영향을 주지 않습니다.

***

### 6️⃣ Observer Cheat Sheet

| 목적      | 추천 Observer                              | 특징                      |
| ------- | ---------------------------------------- | ----------------------- |
| 일반 값 처리 | `subscribe(onNext:)`                     | 가장 자주 사용                |
| UI 바인딩  | `bind(to:)`, `Binder`                    | 에러 무시, MainScheduler 고정 |
| 커스텀 로깅  | `subscribe(on:)` (Event)                 | 모든 이벤트 세부 확인            |
| 상태 공유   | `BehaviorSubject`, `ReplaySubject` (Hot) | 구독 즉시 최신 값 전달           |

***

### 7️⃣ Mini Quiz

1. `Binder`와 `AnyObserver`의 두 가지 차이점은?
2. 동일 Observable에 `observe(on: Main)` 없이 바로 `subscribe`하면 어떤 문제 위험?
3. 네트워크 요청 Observable에서 로딩 토글을 구현할 때 `do(onSubscribe:onDispose:)` 대신 `activityIndicator` 오퍼레이터(라이브러리) 사용 장점은?

<details>

<summary>정답</summary>

1. **Binder vs AnyObserver**

* `Binder`는 _MainScheduler 강제_ & _에러를 무시_, `AnyObserver`는 자유로운 스레드 & 에러 전달 가능.
* `Binder`는 `Reactive` extension 컨텍스트에서만 생성, `AnyObserver`는 어디서든 사용.

2. UI 변경을 백그라운드 스레드에서 수행해 **크래시**(`UIView` 사용 비-main) 위험. `observe(on: MainScheduler.instance)` 또는 `bind(to:)` 필요.
3. `activityIndicator` 오퍼레이터는 재사용 가능하고 _Sync lock_ 문제를 피하면서 스트림 체이닝만으로 로딩 상태를 관리해 **보일러플레이트 감소**.

</details>
