# PublishSubject

## 📢 PublishSubject — 가장 기본이 되는 Hot Subject

> **핵심 키워드:** _구독 이후_ 이벤트만 전달 · 초기값 없음 · Completed/Error 후 자체 소멸

`PublishSubject<Element>`는 **Cold Observable**과 **Observer** 양쪽 역할을 모두 수행하는 _Hot_ 스트림입니다. 구독 시점 이전 이벤트는 전달하지 않기 때문에 **알림(Notification) 패턴**에 적합합니다.

***

### 1️⃣ 특징 한눈에 보기

| 항목       | 설명                                                           | 메모                        |
| -------- | ------------------------------------------------------------ | ------------------------- |
| 초기값      | ❌ 없음                                                         | 필요하면 `BehaviorSubject` 고려 |
| 캐시       | ❌ 전달된 값 저장 X                                                 | `ReplaySubject` 참조        |
| Hot/Cold | **Hot**                                                      | 구독 이전 이벤트 손실 가능           |
| 완료 이벤트   | `.onCompleted()` / `.onError()` 발생 시 **스트림 종료 & 모든 구독자에 전파** |                           |
| 메모리      | 완료 후 내부 버퍼 해제                                                | 주입 값 누수 위험 낮음             |

***

### 2️⃣ 기본 사용법

```swift
let subject = PublishSubject<String>()
let bag = DisposeBag()

// 1st 구독자
subject
    .subscribe(onNext: { print("🅰️:",$0) })
    .disposed(by: bag)

subject.onNext("Hello")

// 2nd 구독자 (이 시점 이전 값 받지 못함)
subject
    .subscribe(onNext: { print("🅱️:",$0) })
    .disposed(by: bag)

subject.onNext("RxSwift")
subject.onCompleted()
```

**출력**

```
🅰️: Hello
🅰️: RxSwift
🅱️: RxSwift
```

* 두 번째 구독자는 "Hello"를 받지 못함 (Hot 특성).
* `onCompleted()` 이후엔 새로운 이벤트가 전달되지 않으며, 추가 구독 시 `.completed` 즉시 수신.

***

### 3️⃣ UI & ViewModel 패턴 예제 — 버튼 탭 이벤트 브로드캐스팅

```swift
final class MenuViewModel {
    // 외부에 노출할 readonly Observable
    let itemSelected: Observable<Int>

    private let itemSelectedSubject = PublishSubject<Int>()
    private let bag = DisposeBag()

    init(input: (buttonTap: Observable<Int>)) {
        self.itemSelected = itemSelectedSubject.asObservable()

        input.buttonTap
            .bind(to: itemSelectedSubject) // View ▶︎ ViewModel
            .disposed(by: bag)
    }
}
```

* 여러 구독자(네비게이션, 로그 모듈 등)가 `itemSelected`를 독립적으로 처리 가능.
* 이전 버튼 탭 이벤트를 재전송할 필요가 없으므로 `PublishSubject`가 적합.

***

### 4️⃣ 마블 다이어그램 비교

| Subject            | 구독 이전 값     | 초기값 | 캐시 크기     |
| ------------------ | ----------- | --- | --------- |
| **PublishSubject** | ❌           | ❌   | 0         |
| BehaviorSubject    | ⭕ (최신 1개)   | ⭕   | 1         |
| ReplaySubject(3)   | ⭕ (최근 3개)   | ❌   | n         |
| AsyncSubject       | ❌ (완료 시 1개) | ❌   | 1 (완료 시점) |

***

### 5️⃣ 메모리 & 쓰레드 주의사항

1. **retain cycle**: `subject.onNext(self)` 같은 self 캡처 주의 → `[weak self]` 패턴 권장.
2. **완료 or 에러 후 해제**: 필요 없다면 `subject.onCompleted()` 호출해 메모리 해제 시점 명시.
3. **MainScheduler 보장**: UI 값을 전달할 땐 `observe(on: MainScheduler.instance)` 선행.

***

### 6️⃣ 자주 하는 실수 & 해결책

| 실수                 | 증상                 | 해결                                                 |
| ------------------ | ------------------ | -------------------------------------------------- |
| 구독 전에 이벤트 발행       | 초기 구독자가 값 못 받음     | 구독 → 이벤트 순서 확인 또는 `BehaviorSubject` 사용             |
| `onError` 남발       | 모든 구독 종료, 앱 상태 불안정 | `catchError`로 에러 변환 후 `.onNext(Result.failure)` 패턴 |
| `DisposeBag` 범위 오류 | 스트림 계속 살아있음        | Bag을 VC·VM 프로퍼티로 선언, deinit 로그 확인                  |

***

### 7️⃣ Mini Quiz

1. `PublishSubject`에 구독자가 없을 때 `onNext`를 호출하면 어떻게 될까?
2. `.onError` 호출 뒤 `onNext`를 다시 보내면?
3. 동일 기능 구현 시 `PassthroughSubject`(Combine)와 다른 점 한 가지?

<details>

<summary>정답</summary>

1. **구독자 없음** → 이벤트가 **바로 폐기**되고 메모리 누수는 없다.
2. `onError`로 스트림이 종료되어 **무시**된다(전달되지 않음).
3. Combine의 `PassthroughSubject`는 **completion 강제 타입매개변수 없음** `Failure == Never` 설정 가능, RxSwift는 `Error`가 고정 타입.

</details>

***

> 다음 문서 ▶️ **BehaviorSubject** 로 이동하여 _최신 값 보존_ 전략을 학습해 보세요. 🚀
