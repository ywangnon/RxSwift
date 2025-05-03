# Swift Extension Cheat Sheet

## 📝 RxSwift Extension Cheat Sheet — 실무에 바로 쓰는 유틸 모음

> **“Rx 오퍼레이터가 부족할 때, 우리만의 한 줄을 더!”**
>
> 아래 확장들은 모두 **Observable / Single / Driver / ControlEvent** 등 Rx 타입을 다룹니다. 순수 Swift 유틸(문자열, 배열 등)은 제외했으니, Rx 흐름에서 바로 복사·붙여넣고 사용해 보세요.

***

### 1️⃣ Observable ⟶ Driver/Signal 변환

```swift
import RxSwift
import RxCocoa

extension ObservableType {
    /// 에러를 무시하고 Completed 처리하여 Driver로 변환
    func asDriverOnErrorJustComplete() -> Driver<Element> {
        asDriver { _ in .empty() }
    }

    /// 에러 발생 시 기본값으로 대체한 뒤 Driver 반환
    func asDriver(onErrorJustReturn value: Element) -> Driver<Element> {
        asDriver { _ in .just(value) }
    }
}
```

| 메서드                             | 사용 예                              |
| ------------------------------- | --------------------------------- |
| `asDriverOnErrorJustComplete()` | ViewModel Output → UI 바인딩 시 오류 무시 |
| `asDriver(onErrorJustReturn:)`  | 네트워크 실패 시 Placeholder 데이터 전달      |

***

### 2️⃣ map / filter 계열 Sugar

```swift
extension ObservableType {
    /// 모든 요소를 Void로 변환 (값이 필요 없을 때)
    func mapToVoid() -> Observable<Void> { map { _ in } }
}

extension Observable where Element: OptionalType {
    /// nil 을 필터링하고 래핑을 벗겨낸다 (Optional 제거)
    func filterNil() -> Observable<Element.Wrapped> {
        compactMap { $0.asOptional }
    }
}
```

> `OptionalType` 프로토콜은 `associatedtype Wrapped` 와 `var asOptional: Wrapped?` 로 정의 후 사용.

***

### 3️⃣ ControlEvent 편의

```swift
extension ControlEvent where Element == Void {
    /// 버튼 연타 방지: 기본 300ms 쓰로틀
    func throttleTap(_ interval: RxTimeInterval = .milliseconds(300)) -> ControlEvent<Void> {
        throttle(interval, scheduler: MainScheduler.instance)
    }
}
```

* `button.rx.tap.throttleTap()` 으로 간단 적용

***

### 4️⃣ Subject / Relay Helper

```swift
import RxRelay

extension PublishRelay {
    /// 현재 쓰레드에서 안전하게 onNext (Main 여부 선택)
    func acceptOnMain(_ element: Element) {
        if Thread.isMainThread {
            accept(element)
        } else {
            DispatchQueue.main.async { self.accept(element) }
        }
    }
}
```

***

### 5️⃣ Disposable & DisposeBag

```swift
extension Disposable {
    /// DisposeBag 이 없는 짧은 스코프에서 사용하기 위한 Auto-dispose
    func disposed(by bag: inout [Disposable]) {
        bag.append(self)
    }
}
```

```swift
var tempBag: [Disposable] = []
observable.subscribe().disposed(by: &tempBag)
```

***

### 6️⃣ withUnretained Sugar (iOS 13 이하 지원)

```swift
extension ObservableType {
    /// RxSwift 6 의 withUnretained 를 iOS 12 프로젝트에서도 사용 가능하게 shim.
    func withUnretained<T: AnyObject>(_ owner: T) -> Observable<(T, Element)> {
        map { [weak owner] element in
            guard let owner = owner else { throw RxError.noElements }
            return (owner, element)
        }
        .catchError { _ in .empty() }
    }
}
```

***

### 7️⃣ Mini Quiz (Self‑Check)

1. `mapToVoid()` 와 `ignoreElements()` 의 차이는?
2. Relay 에서 `acceptOnMain` 이 필요한 이유는 무엇일까?
3. `asDriver(onErrorJustReturn:)` 사용 시 무한 루프가 생길 수 있는 시나리오는?

<details>

<summary>Answers</summary>

1. `mapToVoid()` 는 **각 onNext 값을 Void로 변환**하여 이벤트를 그대로 유지, `ignoreElements()` 는 **onNext 자체를 무시**하고 Completion/Error 만 전달.
2. 백그라운드 스레드에서 `accept` 호출 시 **UI Relay** 가 메인 스레드 제약을 어길 위험, 스레드 안전 확보.
3. 기본값을 다시 연산에 사용해 같은 에러가 반복 → `retry` 와 결합 시 주의.

</details>

***

> 모든 확장은 **Rx 중심** 으로 재구성했습니다. 필요에 따라 회사/프로젝트에 맞게 이름·네임스페이스를 조정해 활용해 보세요. 🚀
