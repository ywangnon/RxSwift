# Disposables

## 🗑️ Disposables — 메모리 누수 없는 RxSwift 사용법

> **“구독은 곧 리소스.”** 해지(Dispose)하지 않으면 Observer·Observable·Closure 모두 메모리에 남습니다.

이 장에서는 `Disposable` 프로토콜과 `DisposeBag`, 그리고 자동 해제 오퍼레이터를 통해 _누수 없는_ Rx 코드를 작성하는 방법을 다룹니다.

***

### 1️⃣ Disposable 기본 개념

```swift
public protocol Disposable {
    func dispose()
}
```

* **역할**: Observable 구독을 취소하고 관련 리소스를 해제.
* **생성 경로**: `subscribe()` 메서드는 항상 `Disposable`을 반환.

```swift
let disposable = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .subscribe(onNext: { print($0) })

// 5초 후 수동 해제
DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
    disposable.dispose()
}
```

> _주의_: 수동 `dispose()` 호출을 잊으면 **Hot Observable**과 Timer 등이 영원히 살아있어 누수·배터리 소모 가능.

***

### 2️⃣ DisposeBag — 수집 후 일괄 dispose

| 특징        | 설명                                                    |
| --------- | ----------------------------------------------------- |
| 스코프 단위 관리 | Bag이 deinited 되면 내부 모든 Disposable 자동 해제               |
| 일반 패턴     | ViewController·ViewModel에 `let bag = DisposeBag()` 생성 |
| ARC 활용    | 클로저 retain cycle 없이 생명주기 일치                           |

```swift
class LoginViewModel {
    private let bag = DisposeBag()

    func bind(input: Input) {
        input.loginTap
            .flatMapLatest(api.login)
            .subscribe()
            .disposed(by: bag) // ✅
    }
}
```

***

### 3️⃣ 기타 Disposable 구현체

| 구현체                     | 용도                                              |
| ----------------------- | ----------------------------------------------- |
| **SerialDisposable**    | 내부 Disposable을 _교체_ 가능 (예: 재시도 시 이전 요청 취소)      |
| **CompositeDisposable** | 여러 Disposable을 묶어 일괄 해제 (`DisposeBag` Swift 버전) |
| **RefCountDisposable**  | 참조 카운트 기반, 마지막 dispose 때 실제 해제                  |

```swift
let serial = SerialDisposable()

searchQuery
    .flatMapLatest(api.search) // 새 검색마다 이전 요청 out
    .subscribe()
    .disposed(by: serial)
```

***

### 4️⃣ 자동 해제 오퍼레이터

| 오퍼레이터           | 설명                                  | 예시                    |
| --------------- | ----------------------------------- | --------------------- |
| `take(_:)`      | n개 요소 후 completed                   | 버튼 첫 1회 이벤트 처리        |
| `takeUntil(_:)` | Trigger Observable emit 시 completed | ViewWillDisappear에 연동 |
| `timeout(_:)`   | 기한 내 이벤트 없으면 error                  | 서버 응답 대기              |
| `single()`      | 첫 요소 + completed                    | 결과 1개 보장 API          |

```swift
viewWillDisappearSignal
    .subscribe(onNext: { _ in print("Bye") })
    .disposed(by: bag)

observable
    .takeUntil(viewWillDisappearSignal)
    .subscribe()
    .disposed(by: bag) // 자동 종료
```

***

### 5️⃣ 메모리 누수 체크 팁

1. **Debug Memory Graph**(`⇧⌘I`) 후 ViewController 인스턴스 확인.
2. Xcode `Allocations` Instrument에서 Live Count 추적.
3. RxSwiftCommunity의 **LeakDetector** 사용 (`rx.disableLeakDetection = false`).

***

### 6️⃣ Best Practices Checklist ✅

* [ ] &#x20;모든 `subscribe` 체인 끝에 **`.disposed(by:)`** 또는 `take*`류 오퍼레이터 존재 확인
* [ ] &#x20;VC 해제 시점 테스트: `deinit { print("deinit VC") }`
* [ ] &#x20;**Bind UI** 시 `Driver`, `Signal` 등 _에러·스레드 안전_ 시퀀스 사용
* [ ] &#x20;Long‑lived 흐름(BLE, WebSocket)은 **ViewModel 레벨**에서 관리

***

### 7️⃣ Mini Quiz

1. `DisposeBag` 대신 `SerialDisposable`이 더 적합한 시나리오는?
2. `take(1)`과 `single()`의 동작 차이 한 줄 요약.
3. ViewController deinit이 호출되지 않을 때 확인해야 할 세 가지 항목은?

<details>

<summary>정답</summary>

1. **시나리오**: 사용자가 검색어를 빠르게 바꿀 때 이전 네트워크 요청을 취소하고 최신 요청만 유지해야 하는 경우; `SerialDisposable().swap(_:)`으로 이전 구독 dispose.
2. `take(1)`은 _첫 next 이벤트_ 후 **completed**, `single()`은 _첫 next 이벤트_ 후 **completed 또는 error(요소 >1 또는 0개)**.
3. 확인 항목

* 클로저·Observable에서 self를 **strong 캡처** 했는지 (`[weak self]`).
* **Delegate**·Notification 등 전통적 리테인 사이클.
* `DisposeBag`이 **VC 프로퍼티**가 아닌 전역·싱글턴에 포함되었는지.

</details>
