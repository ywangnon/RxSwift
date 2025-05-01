# Memory Management

## 🧹 Memory Management — DisposeBag 패턴 & Leak 예방

> **“Rx는 clean‑up이 전부다.”**

Reactive 스트림은 강력하지만, **구독 해제(Dispose)가 누락**되면 메모리 누수·배터리 소모·예상치 못한 콜백이 이어집니다. 이 문서에서 DisposeBag 활용, Retain Cycle 차단, Leak 탐지 도구를 정리합니다.

***

### 1️⃣ DisposeBag 101

```swift
class MyViewController: UIViewController {
    private let bag = DisposeBag()

    override func viewDidLoad() {
        super.viewDidLoad()

        viewModel.output
            .observe(on: MainScheduler.instance)
            .bind(to: label.rx.text)
            .disposed(by: bag) // ✅
    }
}
```

* **원리**: `DisposeBag`이 deinit 되면 내부 Disposables의 `dispose()` 자동 호출.
* **스코프**: VC·ViewModel별 1개, 테스팅 시 `TestDisposeBag()` 재사용.

#### resetBag 패턴

```swift
var bag = DisposeBag()

func resetRx() { bag = DisposeBag() } // 모든 구독 일괄 해제
```

***

### 2️⃣ Retain Cycle 방지 — `[weak self]`

```swift
button.rx.tap
    .withUnretained(self) // RxSwift 6 util
    .subscribe(onNext: { vc, _ in
        vc.performAction()
    })
    .disposed(by: bag)
```

* **withUnretained(self)**: 튜플(weakSelf, value) 반환 → 언랩 필요 X.
* 전통 구문: `.subscribe { [weak self] _ in self?.action() }`.
* Subject·Relay 내부에 `self` 저장 금지.

***

### 3️⃣ Leak 탐지 툴

| 도구                            | 특징                          | 사용법                                  |
| ----------------------------- | --------------------------- | ------------------------------------ |
| **Xcode Memory Graph**        | 실시간 인스턴스 그래프                | Debug ▸ View Memory Graph (`⇧⌘I`)    |
| **Instruments ‑ Allocations** | 라이브 객체 카운트                  | Filter by ClassName, Track Live      |
| **RxLeakDetector**            | RxCommunity · 자동 Dispose 검사 | `RxSwiftExt+LeakDetector` 설치 후 로그 확인 |
| **WeakProxy**                 | Timer/Delegate Retain cut   | `WeakProxy(target)` 패턴               |

***

### 4️⃣ Long‑lived Streams 관리

| 스트림                       | 권장 관리 방식                                           |
| ------------------------- | -------------------------------------------------- |
| **Interval/Timer**        | `.takeUntil(dealloc)` or bag dispose               |
| **BLE / WebSocket**       | ViewModel Scope + `Subject` stop on app background |
| **NotificationCenter.rx** | `.takeUntil(self.rx.deallocated)` 자동 구독 해제         |

***

### 5️⃣ Common Smells & Fixes

| Smell                    | 증상                    | 해결                                    |
| ------------------------ | --------------------- | ------------------------------------- |
| Global DisposeBag        | 객체 deinit 후에도 이벤트 수신  | Bag scope를 객체로 제한                     |
| Nested subscribe         | 중첩 클로저로 다중 dispose 필요 | `flatMap` / Operator chain 이용         |
| share(scope:.forever) 남발 | Hot 공유가 앱 종료까지 유지     | scope 검토 or manual connect/disconnect |

***

### 6️⃣ Mini Checklist ✅

* [ ] &#x20;각 `subscribe` 체인 끝에 `.disposed(by:)` 또는 자동 완료 오퍼레이터
* [ ] &#x20;`[weak self]` or `withUnretained(self)` 사용
* [ ] &#x20;ViewModel → View 방향 데이터, View → ViewModel 입력 Subject 캡슐화
* [ ] &#x20;Memory Graph에서 VC 인스턴스가 화면 전환 후 해제 확인

***

### 7️⃣ Mini Quiz

1. `PublishSubject`를 뷰컨트롤러 프로퍼티로 보관하면 VC가 deinit되어도 스트림이 살아있을까?
2. `Driver` 사용 시 DisposeBag이 필요 없는 경우가 있을까?
3. Xcode Memory Graph에서 Retain Cycle을 확인하는 세 단계는?

<details>

<summary>Answers</summary>

1. Subject 자체가 **Strong reference**를 유지하므로 구독자가 없어도 살아있음; VC가 Bag과 함께 Subject를 nil로 만들어야 해제.
2. Driver는 내부적으로 `share(replay:1)` + `observeOn(Main)`이라도 **구독 해제 시 dispose 필요**; 일반적으로 Bag으로 관리, 하지만 단발 `.drive()` 후 `.asDriver(onErrorJustReturn:)` 완료 스트림은 자동 해제.
3. ① **Debug ▸ View Memory Graph** ② 왼쪽 Inspector에서 **Cycles & Roots** 선택 ③ 관심 클래스 필터 → Reference Trace 확인.

</details>

***

> 다음 ▶️ **patterns.md** 로 이동해 MVVM, Coordinator 등 아키텍처와 Rx 통합 전략을 살펴봅니다. 🚀
